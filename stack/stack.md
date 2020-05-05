
[toc]

[main page](../entry.md)

* [cs courses](https://exploredegrees.stanford.edu/schoolofengineering/computerscience/#masterstext)
* 处理堆栈
    * [统计学习那些事](https://cosx.org/2011/12/stories-about-statistical-learning/)
    * [那些年，我们一起追的 EB](https://cosx.org/2012/05/chase-after-eb/)
    * [正态分布的前世今生（上）](https://songshuhui.net/archives/76501)
    * [正态分布的前世今生（下）](https://songshuhui.net/archives/77386)
    * [CTC——下雨天和RNN更配哦](https://zhuanlan.zhihu.com/p/23308976)
    * [今日头条推荐算法原理首公开，头条首席算法架构师带来详细解读](https://www.leiphone.com/news/201801/XlIxFZ5W3j8MvaEL.html)
    * [数学家（数学专业）都是怎么搞研究的?](https://www.zhihu.com/question/62899869)
    * [清华大学计算机专业本科这位在「自己写的 CPU 上运行自己写的操作系统」的同学是什么水平？](https://www.zhihu.com/question/345718537)

* 会议
    * 出版社
        * IEEE: 出版社，接稿质量高，期刊比会议质量高，旗下有较多子期刊和会议
        * ACM:出版社，接稿质量高，旗下有较多子期刊和会议 https://dl.acm.org/events.cfm
        * Springer:出版社
    * 数据库
        * SCI:数据库，只收录期刊，收录质量高，收录IEEE/ACM出版社的大部分期刊，1区~4区等级由中科院划分
            * [刊物级别查询](http://www.letpub.com.cn/index.php?page=journalapp&view=search)
        * EI:数据库，收录质量相对较低
        * Nature:数据库，收录质量极高，科学家级别
        * Science:数据库，收录质量极高，科学家级别
    * 顶会分级
        * CCF（中国计算机学会）将计算机方向细分为10个子方向，按照A/B/C对期刊和会议分级，可供参考，[中国计算机学会推荐国际学术会议和期刊目录](https://www.ccf.org.cn/Academic_Evaluation/By_category/)
        * [CSRanking](http://csrankings.org/#/index?all)，划分更细 

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
* 数学
* Matrix system equation
	Krylov space https://www.zhihu.com/question/23309010
	Galerkin thm  https://zhuanlan.zhihu.com/p/33562186
	Arnoldi Algorithm https://zhuanlan.zhihu.com/p/33580461
	GMRES Algorithm https://zhuanlan.zhihu.com/p/33616596
	Diff between Arnoldi & GMRES https://www.zhihu.com/question/36746175
	QR converge rate for ill-conditioned matrices
https://sc18.supercomputing.org/proceedings/tech_poster/poster_files/post193s2-file2.pdf

* 变分法
    * Difference between Variation of Calculus problems and Control Theory problems?
    * https://math.stackexchange.com/questions/782621/difference-between-variation-of-calculus-problems-and-control-theory-problems
* 微分几何
    * 圆环可以展成直线，球面为什么就不能展成平面？ 来自 <https://www.zhihu.com/question/295692434> 
    * John Morgan:黎曼几何、曲率、Ricci流以及在三维流形上的应用二讲  来自 <https://www.cnblogs.com/misaka01034/p/JMRiemannGeometry.html> 
    * 【理解黎曼几何】5. 黎曼曲率  来自 <https://www.spaces.ac.cn/archives/4014> 
    * 曲率 来自 <https://baike.baidu.com/item/%E6%9B%B2%E7%8E%87> 

* PCA
目标一，最大化投影矩阵方差，max(wx)(w为对应向量方向)，约束条件为||w||=1，可转换为max(wSw)，S=(1/n)sum(xx)，最终得到lamda=最大特征值(wSw=wλw=λ)
即cross-correlation=E[(X-ux)(Y-uy)]
目标二，最小化降维误差，min||x'-x||2，也可转换为最小化(1/n)sum(u'xx'u)=sum(uSu')，即得到最小特征值以及对应的特征向量
其他，线性变换。X=AA'  A=VΛU'   X=VΣV'  ΛΛ=Σ  保持不变:R=V'A=V'VΛU'=ΛU' 降维：R=Va'A=Va'VΛU'
详见-主成分分析PCA算法：为什么去均值以后的高维矩阵乘以协方差矩阵的特征向量矩阵就是"投影矩阵"