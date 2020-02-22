
[toc]

[system](./system.md)

# AI集群框架

## 涉及的问题

* 数据传输带宽
  * 单机内传输
    * PCI-E，NVLINK
  * 多机间传输
    * 以太网，Inifiniband，Omini-Patch Architecture
* 文件读写带宽
  * 随机读写-IOPS，大文件读写-带宽
  * 介质：HDA，SSD，3d xpoint
  * refer [storage performance](./system.md#performance)
* 并行方案
  * 方案
    * 数据并行：计算节点单独进行前向与反向运算，梯度规约之后进行更新
    * 模型并行：不同的计算节点负责网络的一部分运算
    * 混合并行：数据并行+模型并行
  * 常用策略
    * 由于数据并行虽然加速明显，但需要节点间数据传输；因此计算密集型操作可数据并行，数据密集型操作可模型并行。比如conv适合数据并行，fc适合模型并行

# AI嵌入式框架

# 大数据框架

* 分布式计算框架
  * MapReduce:离线批处理框架
  * Tez:DAG计算框架
  * Spark:迭代/内存计算框架
  * Storm:实时流计算框架
* 技术支撑
  * 一致性算法：Paxos
  * 分布式协同：Apache Zookeeper & Google Chubby
  * 分布式存储：HDFS & HBase