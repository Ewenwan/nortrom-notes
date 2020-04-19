
[toc]

[main page](../entry.md)

# build

## windows

* windows环境下的LIB/LIBPATH/INCLUDE/PATH的配置
    * LIB/LIBPATH的区别
        * LIB is for the linker, helps it find import and static libraries.
        * LIBPATH is for the compiler, helps it find metadata files. Like type libraries, .NET assemblies, WinRT .winmd files.
        * 参考：[What is the difference between the LIB and LIBPATH environment variables for MS Visual C/C++?](https://stackoverflow.com/questions/20483619/what-is-the-difference-between-the-lib-and-libpath-environment-variables-for-ms)
* windows的交叉编译工具链
    * CL.exe和link.exe均为visual studio提供的工具链，在vc/bin中可以找到不同的交叉工具链
    * Specifies a path that the linker will search before it searches the path specified in the LIB environment option.
    * [Use the Microsoft C++ toolset from the command line](https://docs.microsoft.com/en-us/cpp/build/building-on-the-command-line?view=vs-2017)
* windows kits & visual studio 关联

| Visual Studio | Windows Kits | Microsoft Visual Studio |
|--------------------|-------------------|-------------------------------|
| Visual Studio 2013 | Windows Kits 8\.1 | Microsoft Visual Studio 12\.0 |
| Visual Studio 2015 | Windows Kits 8\.1 | Microsoft Visual Studio 14\.0 |
| Visual Studio 2017 | Windows Kits 10   | Microsoft Visual Studio/2017  |

* windows各运行时库
    * MD链接动态库，优势与linux中的.so类似
    * 多库编译时最好保证库的链接模式一致，否则会出现如下错误：
        * `LNK2038: mismatch detected for 'RuntimeLibrary': value 'MT_StaticRelease' doesn't match value 'MD_DynamicRelease' in file.obj`

| type                      | build option | lib name | enabled macros     |
|---------------------------|--------------|----------|--------------------|
| Single threaded           | /ML          | LIBC     | \(none\)           |
| Static multiThread        | /MT          | LIBCMT   | \_MT               |
| Dynamic Link\(DLL\)       | /MD          | MSVCRT   | \_MT \_DLL         |
| Debug Single Thread       | /MLd         | LIBCD    | \_DEBUG            |
| Debug Static MultiThread  | /MTd         | LIBCMTD  | \_DEBUG \_MT       |
| Debug Dynamic Link\(DLL\) | /MDd         | MSVCRTD  | \_DEBUG \_MT \_DLL |


## linux

* `C_INCLUDE_PATH`
    * for C header files
* `CPLUS_INCLUDE_PATH`
    * for C++ header files
* `PATH`
    * executables
* `PYTHONPATH`
    * Python libraries
* `LIBRARY_PATH`
    * is used by gcc before compilation to search directories containing static and shared libraries that need to be linked to your program.
* `LD_LIBRARY_PATH`
    * is used by your program to search directories containing shared libraries after it has been successfully compiled and linked.

# makefile

## 基本语法

* 常见符号
    * \$\@  表示目标文件
    * \$\^  表示所有的依赖文件
    * \$\<  表示第一个依赖文件
    * \$\?  表示比目标还要新的依赖文件列表
    * =赋值
        * `=` 是最基本的赋值
        * `:=` 是覆盖之前的值
        * `?=` 是如果没有被赋值过就赋予等号后面的值
        * `+=` 是添加等号后面的值
        * 以下面例子进行说明
        * =  B = $(A)时，虽然该语句之前A没有定义，但是在其后定义了，所以能输出later
        * := B :=$(A)时，它只会到这句语句之前去找A的值，因A没有定义所以什么都没有输出。
        
        ```
        B = $(A) 
        A = later
        all:
            @echo $(B)
        ```

* target-prerequeisites-command  
    
    ```
    target ... : prerequisites ... 
        command 
    ```

    target也就是一个目标文件，可以是Object File，也可以是执行文件。还可以是一个标签（Label）这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说，prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。
* 默认执行
    * 系统只默认执行第一个target，如果没有输入参数
* command的写法
    * command必须由tab启头
    * command若想不可见，在前面加@
    * command若想错误也继续执行，在前面加-
    * ``cd /home/hchen; pwd`` 中间加; 才能使第二个命令正确输出
* 静态依赖
    ```
    target ... : target-pattern: prerequisites ... 
        commands
    ```

    ...targets定义了一系列的目标文件，可以有通配符。是目标的一个集合。 target-pattern是指明了targets的模式，也就是的目标集模式。 prereq-patterns是目标的依赖模式，它对target-parrtern形成的模式再进行一次依赖目标的定义。 
* PHONY
    * 所谓的PHONY这个单词就是伪造的意思，makefile中将.PHONY放在一个目标前就是指明这个目标是伪文件目标，如下：.PHONY:clean这里clean目标没有依赖文件，如果执行make命令的目录中出现了clean文件，由于其没有依赖文件，所以它永远是最新的，所以根据make的规则clean目标下的命令是不会被执行的。
    * PHONY的作用是告诉make这个target不是真正的文件，只是一个虚拟的target。


## 特殊语法

* 仅存在target: dependency的结构
    * 很多场合会出现<target: dependency>而不带command的情况，其实只是command当时并没有给出，在makefile的其他位置给出了command。在target不被指定为目标的时，dependency中的代码并不被执行。
    * 参见build/core/binary.mk中的

    ```
    $(LOCAL_INTERMEDIATE_TARGETS): PRIVATE_CFLAGS := $(LOCAL_CFLAGS)
    ```
* .c到.o
    * 若有特殊情况，可单独对.o的编译进行干预下列方式的干预，但往往在配置好CFLAGS后，这个过程是自动的
    
    ```makefile
    %.o: %.s
		$(CC) -c $< $(CFLAGS) -o $@
    ```

    * 切记不可写成下述模式，否则每次编译的.o都来自于同一个sourcefile
    
    ```makefile
    $(OBJS): $(SOURCES)
        $(CC) -c $< $(CFLAGS) -o $@
    ```

    * -c后面接source file
    * -o后面接输出，可以是exe输出，后面跟binary file，或者直接是.o输出

    ```makefile
    $(CC) $(CFLAGS) -o $(TARGET) $(OBJS) ../vecmath_lib.a
    $(CC) -c $< $(CFLAGS) -o $@
    ```

## 函数

* makefile里的函数
    * makefile里的函数使用，和取变量的值类似，是以一个‘$’开始，然后是一个括号里面是函数名和需要的参数列表，多个变量用逗号隔开，像这样
    * `return = $(functionname  arg1,arg2,arg3...)`
* call
    * call函数是唯一一个可以用来创建新的参数化的函数。你可以写一个非常复杂的表达式，这个表达式中，你可以定义许多参数，然后你可以用call函数来向这个表达式传递参数。其语法是：

        ```
        $(call <expression>,<parm1>,<parm2>,<parm3>...)
        ```

    * 当 make执行这个函数时，``<expression>``参数中的变量，如``$(1)，$(2)，$(3)``等，会被参数``<parm1>，<parm2>，<parm3>``依次取代。而``<expression>``的返回值就是 call函数的返回值。例如下面，那么，foo的值就是“a b”。

        ```
        reverse = $(1) $(2)
        foo = $(call reverse,a,b)
        ```

* wildcard
    * 使用：`SRC = $(wildcard *.c ./foo/*.c) `
    * 搜索当前目录及./foo/下所有以.c结尾的文件，生成一个以空格间隔的文件名列表，并赋值给SRC.当前目录文件只有文件名，子目录下的文件名包含路径信息，比如./foor/bar.c
* notdir
    * 使用：`SRC = $(notdir wildcard)`
    * 去除所有的目录信息，SRC里的文件名列表将只有文件名。
* patsubst
    * 使用：`OBJ = $(patsubst %.c %.o $(SRC))`
    * patsubst是patten substitude的缩写，匹配替代的意思。这句是在SRC中找到所有.c 结尾的文件，然后把所有的.c换成.o。


## 易错点

* 函数编写时，切记逗号之间没有空格，因为空格也是输入
    * 比如``$(call source-to-object arg1,arg2,arg3)``
* 依赖关系声明时，依赖项没有逗号，相互之间使用空格
    * 比如``target: $(def1) $(def2)``
* 赋值表达式中，=两边均可以存在空格
    * ``value := "source" ``和``value:=source``一样


## 问题分析

* undefined symbol
    * analyze symbol first by:
        * ``nm`` command
        * ``objdump -p`` command
    * case 1: [library function name is c++ type not c type](https://stackoverflow.com/questions/9837009/getting-undefined-symbol-error-while-dynamic-loading-of-shared-library)
    * solution: use extern "C" {...} and with "nm" command to analyze symbol
    * case 2: the library you loaded have no information of the library it depends, and the symbol is in that library
    * e.g: lib A depend on lib B, and refer func f in B, load lib A error since it can't find lib B
    * solution: for ``Makefile.am automake`` related tool, better using *.la in LIBADD than *.so.([see here](http://stackoverflow.com/questions/23685981/what-is-the-difference-between-ldadd-and-libadd)), for ``Makefile`` use ``Wl,-rpath`` to find the library path in runtime

# automake

* how to series
    * [how to build a program with a static library](https://www.gnu.org/software/automake/manual/html_node/A-Library.html)
    
    ```
    noinst_LIBRARIES = libcpio.a
    libcpio_a_SOURCES = …

    bin_PROGRAMS = cpio
    cpio_SOURCES = cpio.c …
    cpio_LDADD = libcpio.a
    ```
* makefile.am & configure.in
    * automake流程如下所示，参见[例解 autoconf 和 automake 生成 Makefile 文件](https://www.ibm.com/developerworks/cn/linux/l-makefile/index.html)
        1. 运行autoscan命令
        2. 将configure.scan 文件重命名为configure.in，并修改configure.in文件
        3. 在project目录下新建Makefile.am文件，并在core和shell目录下也新建makefile.a.文件
        4. 在project目录下新建NEWS、 README、 ChangeLog 、AUTHORS文件
        5. 将/usr/share/automake-1.X/目录下的depcomp和complie文件拷贝到本目录下
        6. 运行aclocal命令
        7. 运行autoconf命令
        8. 运行automake -a命令
        9. 运行./confiugre脚本

    ![automake](./data/makefile/makefile.png.gif)

    * makefile.am & makefile.in区别（[What are Makefile.am and Makefile.in?](https://stackoverflow.com/questions/2531827/what-are-makefile-am-and-makefile-in)）
        * Makefile.am is a programmer-defined file and is used by automake to generate the Makefile.in file (the .am stands for automake).
        * The configure script typically seen in source tarballs will use the Makefile.in to generate a Makefile.