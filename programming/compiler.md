
[toc]

[programming](./prog.md)

# GCC

## 编译选项

### 常用选项

* [常用指令](https://gcc.gnu.org/onlinedocs/gcc-9.2.0/gcc/Overall-Options.html#Overall-Options)
* 编译选项
    * `-c`  Compile or assemble the source files, but do not link. 
    * `-S`  Stop after the stage of compilation proper; do not assemble.
    * `-o file`  This applies regardless to whatever sort of output is being produced, whether it be an executable file, an object file, an assembler file or preprocessed C code. 
* 优化选项
    * -O == -O1
    * -O2已经为较激进优化
    * -O3更为激进
    * -Os和-O2有一定的差异性，主要表现在控制生成目标的size（为android默认优化选项）
* 特殊属性(attribute)
    * [属性的语法范式](https://gcc.gnu.org/onlinedocs/gcc/Attribute-Syntax.html)

### 特殊选项

* fomit-frame-point
    * fp record the history stack of outstanding calls. Most smaller functions don't need a frame-pointer
    * larger functions MAY need one. this option allows one extra register to be available for general-purpose use. In thumb it is R7
    * 查看：[Trying to understand gcc option -fomit-frame-pointer](https://stackoverflow.com/questions/14666665/trying-to-understand-gcc-option-fomit-frame-pointer)

## GDB调试

* 状态查看
    * 查看函数堆栈`bt`
    * 查看一次寄存器状态`info registers`
    * 打印变量
        * `p your_variable`
* 控制执行
    * 单步执行(step into)`step & s`
    * 单条语句(step over)`n`
    * 单步汇编指令执行`si` & `stepi`
    * 继续运行`c`
    * 添加断点`b xxx.c:line_num`，比如`b main.c:97`
    * 运行`r`
