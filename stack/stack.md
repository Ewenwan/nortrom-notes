
[toc]

[main page](../entry.md)

* 处理堆栈
    * [统计学习那些事](https://cosx.org/2011/12/stories-about-statistical-learning/)
    * [那些年，我们一起追的 EB](https://cosx.org/2012/05/chase-after-eb/)
    * [正态分布的前世今生（上）](https://songshuhui.net/archives/76501)
    * [正态分布的前世今生（下）](https://songshuhui.net/archives/77386)
    * [CTC——下雨天和RNN更配哦](https://zhuanlan.zhihu.com/p/23308976)
    * [今日头条推荐算法原理首公开，头条首席算法架构师带来详细解读](https://www.leiphone.com/news/201801/XlIxFZ5W3j8MvaEL.html)
    * [数学家（数学专业）都是怎么搞研究的?](https://www.zhihu.com/question/62899869)
    * [清华大学计算机专业本科这位在「自己写的 CPU 上运行自己写的操作系统」的同学是什么水平？](https://www.zhihu.com/question/345718537)

* jingbang学习栈
    * 学习halide & tvm & tiramisu框架
        * halide：https://halide-lang.org/
        * halide get started: https://halide-lang.org/#gettingstarted
        * halide tutorial: https://halide-lang.org/tutorials/tutorial_introduction.html (1-21都要学习)
        * tvm: https://tvm.apache.org/
        * tvm tutorial: 要把这两个doc看了，https://tvm.apache.org/docs/tutorials/tensor_expr_get_started.html#sphx-glr-tutorials-tensor-expr-get-started-py & https://tvm.apache.org/docs/tutorials/optimize/opt_gemm.html#sphx-glr-tutorials-optimize-opt-gemm-py
        * tiramisu: http://tiramisu-compiler.org/
        * tiramisu tutorial: https://github.com/Tiramisu-Compiler/tiramisu/blob/master/tutorials/README.md 看01 ~ 04B
    * 至少需要思考的问题
        * halide是做什么的？
        * halide框架通过什么方式实现了优化？
        * 针对图像算子和矩阵乘法halide框架使用了哪些优化策略？
        * TVM和tiramisu基于halide框架做了哪些改进？两者有何不同？