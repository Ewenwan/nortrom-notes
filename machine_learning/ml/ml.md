[toc]

[main page](../../entry.md)

# 基础模型

# 基本概念

* normalization（归一化）
    * 作用
        * 减少求取最优解的迭代次数
        * 未经归一化和经过归一化后的数据的求取最优解过程如下图：

    <img src="./data/normalization1.png" width=40%> 
    <img src="./data/normalization2.png" width=40%> 

    * 方法
        * Min-Max Normalization（min-max标准化）
        $$
            x^*=\frac{x-Min}{Max-Min}
        $$
            * 缺陷：当有新数据加入时，可能导致max和min的变化，需要重新定义。
        * Z-score Normalization（Z-score标准化）
        $$
            x^8=\frac{x-u}{\sigma}
        $$