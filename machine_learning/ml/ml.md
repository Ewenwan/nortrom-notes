[toc]

[main page](../../entry.md)

# 基础模型

## Logistic Regression

* LR基础
    *  logistic regression是这么假设的： 数据服从概率为p的二项分布，并且logit(p)是特征的线性组合。
    *  采用的损失函数为logistical loss
    *  logistic regression实际上还是一个分类方法，只不过加上了Logistic映射使得它的目标拟合曲线是一个分类曲线（y=1/-1）

## SVM

* SVM和logistic回归的区别
    * 逻辑回归采用的是logistical loss，svm采用的是hinge loss。这两个损失函数的目的都是增加对分类影响较大的数据点的权重，减少与分类关系较小的数据点的权重。SVM的处理方法是只考虑support vectors，也就是和分类最相关的少数点，去学习分类器。而逻辑回归通过非线性映射，大大减小了离分类平面较远的点的权重，相对提升了与分类最相关的数据点的权重。
    * 逻辑回归相对来说模型更简单，好理解，实现起来，特别是大规模线性分类时比较方便。而SVM的理解和优化相对来说复杂一些。但是SVM的理论基础更加牢固，有一套结构化风险最小化的理论基础，虽然一般使用的人不太会去关注。还有很重要的一点，SVM转化为对偶问题后，分类只需要计算与少数几个支持向量的距离，这个在进行复杂核函数计算时优势很明显，能够大大简化模型和计算量。

# 基本概念

* normalization & standardization
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
            * Z-score Normalization（Z-score标准化）-- 也就是standardization
            $$
                x^8=\frac{x-u}{\sigma}
            $$

    * comparison
        * most algorithms will probably benefit from standardization more so than from normalization.
        * image processing: normalization, fit a certain range
        * pca: standardization, share the same distance measure
        * [Linear Regression :: Normalization (Vs) Standardization](https://stackoverflow.com/questions/32108179/linear-regression-normalization-vs-standardization)