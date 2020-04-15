
[toc]

[system](./system.md)

# 关系型数据库

# 非关系型

## redis

* 存储方式
    * key-value存储系统
* 特点
    * 可扩展性；
        * 分布式的架构决定了只要有更多的机器，就能够保证存储更多的数据；
    * 高并发性
        * 主从结构，主设备只负责写任务，从设备负责数据读取，可得到10万QPS级别，相比于关系数据的几千QPS量级，提升明显
* 应用
    * 当前，几乎所有的现有的云存储都是 key-value 形式的，例如 Amazon的 smipleDB，底层实现就是 key-value，还有 google 的 GoogleAppEngine，采用的是 BigTable的存储形式。
* 一致性问题
    * 如何解决？1.等待数据自动过期；2.显示更新database，删除缓存；3.直接访问database
    * [如何保持mysql和redis中数据的一致性？](https://www.zhihu.com/question/319817091)