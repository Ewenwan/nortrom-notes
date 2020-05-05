
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
    * `-O` == -O1
    * `-O2`已经为较激进优化
    * `-O3`更为激进
    * `-Os`和-O2有一定的差异性，主要表现在控制生成目标的size（为android默认优化选项）
    * `-ffast-math`
        * `-ffast-math` does a lot more than just break strict IEEE compliance. First of all, of course, it does **break strict IEEE compliance**, allowing e.g. the reordering of instructions to something which is mathematically the same (ideally) but not exactly the same in floating point. Second,  it **disables setting errno** after single-instruction math functions, which means avoiding a write to a thread-local variable (this can make a 100% difference for those functions on some architectures). Third, it makes the **assumption that all math is finite**, which means that **no checks for NaN (or zero)** are made in place where they would have detrimental effects. It is simply assumed that this isn't going to happen. Fourth, it **enables reciprocal approximations for division and reciprocal square root**. Further, it **disables signed zero** (code assumes signed zero does not exist, even if the target supports it) and **rounding math**, which enables among other things constant folding at compile-time. Last, it generates code that **assumes that no hardware interrupts can happen due to signalling/trapping math** (that is, if these cannot be disabled on the target architecture and consequently do happen, they will not be handled).
        * it includes `-fno-math-errno`, `-funsafe-math-optimizations`, `-ffinite-math-only`, `-fno-rounding-math`, `-fno-signaling-nans`, `-fcx-limited-range` and `-fexcess-precision=fast`
        * 参考：[What does gcc's ffast-math actually do?](https://stackoverflow.com/questions/7420665/what-does-gccs-ffast-math-actually-do)
* 特殊属性(attribute)
    * [属性的语法范式](https://gcc.gnu.org/onlinedocs/gcc/Attribute-Syntax.html)

### 特殊选项

* fomit-frame-point
    * fp record the history stack of outstanding calls. Most smaller functions don't need a frame-pointer
    * larger functions MAY need one. this option allows one extra register to be available for general-purpose use. In thumb it is R7
    * 查看：[Trying to understand gcc option -fomit-frame-pointer](https://stackoverflow.com/questions/14666665/trying-to-understand-gcc-option-fomit-frame-pointer)
* fvisibility(链接选项)
    * 将库中的symbol隐藏
        * `-fvisibility=default|internal|hidden|protected`
    * 可以设置所有符号全部隐藏，但暴露部分符号：
        * 暴露的符号： `__attribute__ ((visibility ("default"))) ` and pass `-fvisibility=hidden` to the compiler
    * 也可以设置所有符号全部暴露，但部分隐藏：
        * 隐藏的符号： `__attribute__ ((visibility ("hidden"))) `
    * 参考 [how-to-hide-the-exported-symbols-name-within-a-shared-library](https://stackoverflow.com/questions/9648655/how-to-hide-the-exported-symbols-name-within-a-shared-library)
* fpic
    * Generate position independent code
    * All objects in a shared library should be fpic or not fpic(keep the same) ([GCC -fPIC option](https://stackoverflow.com/questions/5311515/gcc-fpic-option))
    * diff between `fpic` & `fPIC`
    * fpic and fPIC区别
        * Use -fPIC or -fpic to generate position independent code. Whether to use -fPIC or -fpic to generate position independent code is target-dependent. **The -fPIC choice always works, but may produce larger code than -fpic** (mnenomic to remember this is that PIC is in a larger case, so it may produce larger amounts of code). Using **-fpic option usually generates smaller and faster code, but will have platform-dependent limitations**, such as the number of globally visible symbols or the size of the code. The linker will tell you whether it fits when you create the shared library. When in doubt, I choose -fPIC, because it always works.
	    * `-fpic` Generate position-independent code (PIC) suitable for use in a shared library, if supported for the target machine. Such code accesses all constant addresses through a global offset table (GOT). The dynamic loader resolves the GOT entries when the program starts (the dynamic loader is not part of GCC; it is part of the operating system). If the GOT size for the linked executable exceeds a machine-specific maximum size, you get an error message from the linker indicating that -fpic does not work; in that case, recompile with -fPIC instead. (These maximums are 8k on the SPARC, 28k on AArch64 and 32k on the m68k and RS/6000. The x86 has no such limit.) Position-independent code requires special support, and therefore works only on certain machines. For the x86, GCC supports PIC for System V but not for the Sun 386i. Code generated for the IBM RS/6000 is always position-independent.
* `-rpath`
    * 链接选项，运行期生效
    * Add a directory to the runtime library search path. This is used when linking an ELF executable with shared objects. All -rpath arguments are concatenated and passed to the runtime linker, which uses them to locate shared objects at runtime.
* `-L`
    * 链接选项，编译期链接生效
    * `--library-path=searchdir`
    * Add path searchdir to the list of paths that ld will search for archive libraries and ld control scripts.
    * So, `-L` tells ld where to look for libraries to **link against when linking**. You use this (for example) when you're building against libraries in your build tree, which will be put in the normal system library paths by make install. `--rpath`, on the other hand, stores that path inside the executable, so that the **runtime dynamic linker can find the libraries**. You use this when your libraries are outside the system library search path.
    * 参考：[What's the difference between -rpath and -L?](https://stackoverflow.com/questions/8482152/whats-the-difference-between-rpath-and-l)
* `-Wl`
    * The `-Wl,xxx` option for gcc passes a comma-separated list of tokens as a space-separated list of arguments to the linker. So `gcc -Wl,aaa,bbb,ccc` eventually becomes a linker call `ld aaa bbb ccc`
    * 参考：[I don't understand -Wl,-rpath -Wl,](https://stackoverflow.com/questions/6562403/i-dont-understand-wl-rpath-wl)

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
* 调试脚本
    * ` ./gdbtest  -command=gdbtest.sh`

# transpiler

* all python to cpp transpiler projects
    * [py14](https://github.com/lukasmartinelli/py14)
    * [pythran](https://github.com/serge-sans-paille/pythran)
    * [py2c](https://github.com/pradyun/py2c)
    * [transpyle](https://pypi.org/project/transpyle/#id2)
    * [cython](http://docs.cython.org/en/latest/src/tutorial/cython_tutorial.html)
    * [pyrex](http://www.cosc.canterbury.ac.nz/greg.ewing/python/Pyrex/version/Doc/About.html)
    * [nuikta](https://nuitka.net/doc/user-manual.html)
    * [shedskin](https://github.com/shedskin/shedskin)
* cpython based方案分析
    * **难验证**：基于PyObject以及PyObject衍生出来的系列function缺乏可读性，难以验证生成代码的正确性
    * **兼容性差**： 基于PyObject类型的c代码风格，很难整合其他编译器功能，包括热点分析，热点函数抓取或替换，以及基于模型的代码块分类等。
    * **性能差**： 任何对象，哪怕是简单序列操作都被PyObject化，转化的c代码本身的性能堪忧
* shedskin
    * 框架部署分析
        * 执行环境：需要在嵌入式平台中加入libgc和libpcre3库（[distributing binaries](https://shedskin.readthedocs.io/en/latest/documentation.html#distributing-binaries)）
        * [libgc](https://github.com/ivmai/bdwgc)库，license: FSF(成分较为复杂，需要深入分析)，依赖库：无
        * [libpcre3](http://www.pcre.org/)库，license：BSD，可直接商用，依赖库：无

# TVM

* 图表示
    * NNVM
        * NNVM相当于深度学习领域的LLVM，是一个神经网络中比较高级的中间表示模块，通常称为计算图。前端你框架只需要将其计算表达成NNVM中间表示，之后NNVM则统一的对图做与具体硬件和框架无关的优化。包括内存分配，数据类型和形状的推导，算子融合等。
    * Relay
        * Relay解决了静态图和动态图的矛盾。是一种专用于自动微分编程领域的特定域语言。
    * 图优化方法
        * OpFusion:算子融合
        * FoldConstant:常量折叠
        * CombineParallelConv2D:结合并行的卷积与运算
        * FoldScaleAxis:折叠缩放轴
        * AlterOpLayout:改变算子排布
        * CanonicalizeOps:规范化算子
        * EliminateCommonSubexpr:消除公共子表达式
* 算子优化
    * TVM低层次中间表达的特点
        * Halide & HalideIR
        * Auto-Tuning
        * loopy循环变换工具，多面体模型分析
        * python作为宿主语言