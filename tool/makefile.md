
[toc]

[main page](../entry.md)

# makefile

## 基本语法和函数

* 常见符号
    * \$\@  表示目标文件
    * \$\^  表示所有的依赖文件
    * \$\<  表示第一个依赖文件
    * \$\?  表示比目标还要新的依赖文件列表
    * =赋值
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


## 特殊语法和函数

* 仅存在target: dependency的结构
    * 很多场合会出现<target: dependency>而不带command的情况，其实只是command当时并没有给出，在makefile的其他位置给出了command。在target不被指定为目标的时，dependency中的代码并不被执行。
    * 参见build/core/binary.mk中的

    ```
    $(LOCAL_INTERMEDIATE_TARGETS): PRIVATE_CFLAGS := $(LOCAL_CFLAGS)
    ```
* call函数
    * call函数是唯一一个可以用来创建新的参数化的函数。你可以写一个非常复杂的表达式，这个表达式中，你可以定义许多参数，然后你可以用call函数来向这个表达式传递参数。其语法是：

        ```
        $(call <expression>,<parm1>,<parm2>,<parm3>...)
        ```

    * 当 make执行这个函数时，``<expression>``参数中的变量，如``$(1)，$(2)，$(3)``等，会被参数``<parm1>，<parm2>，<parm3>``依次取代。而``<expression>``的返回值就是 call函数的返回值。例如下面，那么，foo的值就是“a b”。

        ```
        reverse = $(1) $(2)
        foo = $(call reverse,a,b)
        ```


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