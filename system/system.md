
[toc]

[main page](../entry.md)

# link of sub chapter

* [ai_framework](./ai_framework.md)
* [runtime_library](./runtime_library.md)
* [operating_system](./operating_system.md)
* [soc](./soc.md)

# performance

* evolution
    * [evolution of storage and computing](http://pages.experts-exchange.com/processing-power-compared/) -- [localpage](./data/system/Processing%20Power%20Compared.html)
* storage
  * latency of storage in general
      * [Latency Comparison Numbers (~2012)](https://gist.github.com/jboner/2841832)

      ![system_latency.PNG](./data/system/system_latency.PNG)

  * latency of emmc storage

      ![latency of storage](./data/latency_of_each_storage.jpg)

  * 参考文档
    * [3D XPoint、XL-Flash、MRAM：未来鹿死谁手？](https://zhuanlan.zhihu.com/p/78216825)

* PCIE
    * PCIe 吞吐量（可用带宽）计算方法：
        * 吞吐量 = 传输速率 *  编码方案
        * 例如：PCI-e2.0 协议支持 5.0 GT/s，即每一条Lane 上支持每秒钟内传输 5G个Bit；但这并不意味着 PCIe 2.0协议的每一条Lane支持 5Gbps 的速率。为什么这么说呢？因为PCIe 2.0 的物理层协议中使用的是 8b/10b 的编码方案。 即每传输8个Bit，需要发送10个Bit；这多出的2个Bit并不是对上层有意义的信息。那么， PCIe 2.0协议的每一条Lane支持 5 * 8 / 10 = 4 Gbps = 500 MB/s 的速率。以一个PCIe 2.0 x8的通道为例，x8的可用带宽为 4 * 8 = 32 Gbps = 4 GB/s。

    ![pcie.png](./data/pcie.png)

# architecture

## quick question

* hyper threading
  * AMD and ARM has no HyperThreading support
  * why hyper threading: in some cases, there are more cpu idles caused by cache miss, hyper threading save that cpu idle
  * thereafter, hyper threading turns out to have a better throughput in special case.
  * refer [Is HyperThreading / SMT a flawed concept?](https://stackoverflow.com/questions/23078766/is-hyperthreading-smt-a-flawed-concept)