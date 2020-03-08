
[toc]

[operating system](./operating_system.md)

# 性能分析和优化

## 诊断分析

### 诊断工具

* perf
    * 可精确查看cache miss/branch miss/cycles/page-fault/context-switch等数据
    * 其利用硬件PMU和内核性能模块进行性能统计
    * 参考：[Perf -- Linux下的系统性能调优工具，第 1 部分](https://www.ibm.com/developerworks/cn/linux/l-cn-perf1/)
    * 参考：[Perf -- Linux下的系统性能调优工具，第 2 部分](https://www.ibm.com/developerworks/cn/linux/l-cn-perf2/)
* vmstat
    * vmstat命令是最常见的Linux/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。
    * 参考：[Linux vmstat命令实战详解](http://www.cnblogs.com/ggjucheng/archive/2012/01/05/2312625.html)
* 其他调试命令汇总
    * `slabtop`: check current memory usage by analyzing /proc/slabinfo
    * `pmap`: check memory usage of a process
    * `pgrep`: find process by process name
    * `pidof`: find the pid of a process by process name
    * `ps aHs` & `top -H` check thread sts(tgid(by getpid) = main thread pid, * thread pid(by gettid))
    * `iotop`: check current io busy process
    * `iostat`: get current io status

## 性能优化

### 内存优化

* 内存池(memory pool)
    * To **shorten the time that program allocate memory**. Now I show an example to prove the technology have good efficiency
    * To **avoid memory`s fragment**.
* 内存压缩(无损算法)
    * zram -- linux内核配置
        * 使用场景：交换机等设备变化少，内存增益大
        * 使用LZ0等压缩方式
    * [snappy](http://code.google.com/p/snappy/) -- 压缩库
        * 谷歌内部生产环境中被许多项目使用的压缩库，包括BigTable，MapReduce和RPC
        * 纯文本的压缩率为1.5-1.7，对于HTML是2-4
        * 压缩速度快
    * [FastLZ](http://www.quicklz.com/) -- 压缩库
        * 纯文本压缩率为1.9~2.0
    * [LZO/miniLZO](http://www.oberhumer.com/opensource/lzo/) -- 压缩库
        * 开源的无损压缩C语言库，其优点是压缩和解压缩比较迅速占用内存小
    * 参考：[几个常用快速无损压缩算法性能比较](http://blog.sina.com.cn/s/blog_814e83d801019itv.html)
    * 参考：[huffman coding](../programming/algorithm.md#huffman)

# 故障诊断

## 工具

* **yamd**：查找 C 和 C++ 中动态的、与内存分配有关的问题
* **memwatch**：开放源代码 C 语言内存错误检测工具
* **strace**：显示所有由用户空间程序发出的系统调用
* 参考：[掌握 Linux 调试技术](https://www.ibm.com/developerworks/cn/linux/sdk/l-debug/index.html)

# 系统调用

* [各平台系统调研指令](http://man7.org/linux/man-pages/man2/syscall.2.html)
* [mprotect](http://man7.org/linux/man-pages/man2/mprotect.2.html)
    * changes the access protections for the calling process's memory pages containing any part of the address range in the interval
    * 用处：动态设置某个mmap的memory区域为可执行
    * 用法:
        * 1.run code in executable heap:
            * get page size: `sysconf(_SC_PAGE_SIZE)`
            * create a heap region: `memalign(PAGESIZE * x)`
            * set it to executable: `mprotect(ptr, 4 * pagesize, PROT_READ | PROT_WRITE | PROT_EXEC) == -1`
        * 2.run code in a executable vma:
            * `void* mptr = mmap(0,4096,PROT_READ|PROT_WRITE|PROT_EXEC,MAP_PRIVATE|MAP_ANON,-1,0);`
* pread
    * Pread() works just like read() but reads from the specified position in the file without modifying the file pointer.
* `siglongjmp` & `sigsetjmp`  `getcontext` & `setcontext`
    * [`siglongjmp`](http://man7.org/linux/man-pages/man3/siglongjmp.3.html) & `sigsetjmp`
        * The setjmp() function saves various information about the calling environment
        *  The longjmp() function uses the information saved in env to transfer control back to the point where setjmp() was called and to restore ("rewind") the stack to its state at the time of the setjmp() call.
    * [`getcontext`](http://man7.org/linux/man-pages/man3/getcontext.3.html) & `setcontext`
        * The setcontext() function restores the user context pointed to by ucp. A successful call to setcontext() does not return; program execution resumes at the point specified by the ucp argument passed to setcontext().
    * 比较：两者成对出现，具体说明可以参见libc或者bionic，用于保存上下文状态，siglongjmp一组更多用于signal的处理，但两者的用途都是一样的。其中android平台并不支持getcontext。
    * 演进过程： `setjmp` & `longjmp` ---> `siglongjmp` & `sigsetjmp` --->  `getcontext` & `setcontext`
    * 参考：[setcontext wiki](https://en.wikipedia.org/wiki/Setcontext)
* mutex & [futex](http://man7.org/linux/man-pages/man2/futex.2.html)
    * mutex
        * Mutex provide mutual exclusion by having a memory area which can toggle between locked and unlocked atomically. The memory resides in user space but the toggling happens through a system call inside the kernel. The reason why the system call is required (and hence heavy weight) is because:
            * WAKE - The kernel has to schedule the next thread in the wait queue
            * WAIT - All accesses to the lock are guarded and queued through a single point of synchronization - the kernel.
    * futex
        * Futex isn't the same as mutex. It provides **a wait queue technique accessible in the user space**. **Pthread Mutex** in the latest Linux distributions are implemented through Futex logic.
    * 参考：[How different is a futex from mutex - conceptually and also implementation wise?](https://www.quora.com/How-different-is-a-futex-from-mutex-conceptually-and-also-implementation-wise)
    * 参考：futex在android pthread中的应用
