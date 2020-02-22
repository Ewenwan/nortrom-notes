
[toc]

[main page](../entry.md)

# link of sub chapter

* [ai_framework](./ai_framework.md)
* [runtime_library](./runtime_library.md)

# performance

* storage
  * latency of storage
![latency of storage](./data/latency_of_each_storage.jpg)
  * 参考文档
    * [3D XPoint、XL-Flash、MRAM：未来鹿死谁手？](https://zhuanlan.zhihu.com/p/78216825)

# architecture

## quick question

* hyper threading
  * AMD and ARM has no HyperThreading support
  * why hyper threading: in some cases, there are more cpu idles caused by cache miss, hyper threading save that cpu idle
  * thereafter, hyper threading turns out to have a better throughput in special case.
  * refer [Is HyperThreading / SMT a flawed concept?](https://stackoverflow.com/questions/23078766/is-hyperthreading-smt-a-flawed-concept)