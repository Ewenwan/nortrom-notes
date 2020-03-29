
[toc]

[system](./system.md)

# Soc General

## design

* 参考：
    * [轻松读懂移动处理器 CPU微架构全解析](http://www.pcpop.com/article/878268_1.shtml)
* 流水线设计
    * 目的：取指、解码、执行、写回都是放在同一个拍（周期）内顺序完成，此时的 CPI(每指令周期数）基本上是 1，但是这样设计的效率很低：当取指的时候，其余工位都只能瞎瞪眼等开饭，这样的设计也被称作非流水线化执行。
    * e.g.: Cortex-A15、Sandy Bridge 都分别具备 15 级、14 级流水线，而 Intel NetBurst（Pentium 4）、AMD Bulldozer 都是 20 级流水线，它们的工位数都远超出基本的四（或者五）工位流水线设计。**更长的流水线虽然能提高频率，但是代价是耗电更高而且可能会有各种性能惩罚**。
    * pipeline stage list:Sony Cell PPU23IBM PowerPC 717IBM Xenon19AMD Athlon10AMD Athlon XP11AMD Athlon6412AMD Phenom12AMD Opteron15ARM7TDMI(-S) 3ARM7EJ-S 5ARM810 5ARM9TDMI 5ARM1020E 6XScale PXA210/PXA250 7ARM1136J(F)-S 8ARM1156T2(F)-S 9ARM Cortex-A5 8ARM Cortex-A813AVR32 AP7 7AVR32 UC3 3DLX 5Intel P5 (Pentium) 5Intel P6 (Pentium Pro) 14Intel P6 (Pentium III) 10Intel NetBurst (Willamette)20Intel NetBurst (Northwood)20Intel NetBurst (Prescott)31Intel NetBurst (Cedar Mill)31Intel Core14Intel Atom16LatticeMico32 6R4000 8StrongARM SA-110 5SuperH SH2 5SuperH SH2A 5SuperH SH4 5SuperH SH4A 7UltraSPARC 9UltraSPARC T1 6UltraSPARC T2 8WinChip 4LC2200 32 bit 5
* 超标量（指令多发）
    * 常用于SIMD计算，此时数据依赖关联往往不强，流水线无需等待
* 分支（转移）预测
    * 目的：处理器平均六条指令就会遇到一条分支指令，因此流水线设计带来的大部分性能提升优势此时会被丧失掉。
    * 方法：
        * 动态分支预测。记录最近的历史信息，用该信息指导预测（动态分支表需要占用相当可观的芯片面积，但是另一方面来说分支预测对流水线化处理器是相当重要的，所以是物有所值的。）
        * 条件执行指令（predicated instruction）
            * 原先
                * 采用了判定指令后，原来的 5 条指令变成 3 条，避免了两条分支指令，cmp 和 mov 可以并行执行实现 50% 的性能提升，消除了分支预测错误导致的大量误预测惩罚。
                * ARM 从一开始就具备完整的判定指令集

                ```asm
                cmp a, 5 ; a > 5 ?
                ble L1
                mov c, b ; b = c
                br L2
                L1: mov d, b ; b = d
                L2: ...
                ```

            * 优化

                ```asm
                cmp a, 5 ; a > 5 ?
                mov c, b ; b = c
                cmovle d, b ; if le, then b = d
                ```
* 动态调度（乱序执行OoOE）
    * 目的：为了充分利用由于分支以及长时延指令导致的流水线“气泡（停摆）”而浪费的资源，人们引入了乱序执行（OoOE）技术。当出现需要等待某条指令的时候，程序中的指令会被“重排序（Re-Ordered）”，使得其他指令可以被执行。
    * 应用：ARM Cortex-A8、Intel Pentium、Intel Atom（Bonnell 内核）、IBM Cell PPU 都属于顺序执行，它们选择顺序执行的原因主要是为了省电，因为 OoOE 需要大量的晶体管来实现。随着制程的改进，**OoOE 的开销会逐渐淡化变得在某些场合里可行**，因此像 ARM 从 Cortex-A9、Intel 从 Pentium Pro/Atom（Silvermont 内核）都开始采用 OoOE。
* 数据级并行（SIMD）
    * SIMD 就是单指令多数据的缩写，理解起来并不困难，例如执行一条 SIMD 加法指令就能在一个周期里完成 64 条数据流发来的 64 个数字的加法运算。SIMD 的初衷是为了摊薄大量执行单元上的控制单元成本，顺带减少程序的尺寸，因为SIMD 只需要复制一份代码就能开跑，而多核处理器（或者说 MIMD）需要每个内核都复制一份代码和在 cache 上共享多个程序拷贝。
* 存储系统分层结构
    * ARM Cortex-A9为例：
        * L1 cache 是 32-KiB（时延 4 周期）
        * L2 Cache 是 1-MiB，不同大小区段的时延是：
        	* 64 KiB - 128KiB = L1C + L2C = 4 + 19 = 23 周期
        	* 256 KiB - 512 KiB = L1_C + L2_C + TLB_L1 = 4 + 19 + 7 = 30 周期
        	* 1 MiB = L1_C + L2_C + TLB_L1 + TLB_L2 = 4 + 19 + 7 +7 = 37 周期
* cache同步管理
    * [snooping control unit](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0407e/CDDEHDDG.html)，自动管理cache同步
    * AXI advanced extended interface 为 AMBA Advanced Microcontroller Bus Architecture 协议中重要的一部分。
    * The SCU connects one to four Cortex-A9 processors to the memory system through the AXI interfaces. SCU的功能主要为（比方说core 0 和 core 1并发使用ldr等操作时，SCU能有效处理这种对内存的并发操作。）：
        * maintain data cache coherency between the Cortex-A9 processors
        * initiate L2 AXI memory accesses
        * arbitrate between Cortex-A9 processors requesting L2 accesses 
        * manage ACP accesses.


## common sense

* big endian，little endian大端序，小端序
    * Most Significant Bit / Least Significant Bit:二进制中最高/低值的比特，即影响最大/小的那一位
    * big endian是指低地址存放最高有效字节（MSB），而 little endian则是低地址存放最低有效字节（LSB）


# GPU

## cuda

* introduction
    * [cuda c programming guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/#introduction)
    * [practice of cuda programming](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/#abstract)
* data transformation
    * device-2-host&host-2-device
        * depending on the speed of PCI-e
    * virtualization UVM (Unified Memory)
        * 统一地址空间的最大优势其实不是性能，而是让 GPU 编程，特别是 GPGPU 编程变得更加简单。试想未来可以像写 CPU 代码一样写出高性能的、并行的 GPU 代码，而不需要关心如何在 CPU 与 GPU 之间拷贝数据，分配空间。


# ARM

## design

* FCSE(fast context switch extension)
    * 原理：FCSE 通过修改系统中不同进程的虚拟地址，避免在进行进程间切换时造成的虚拟地址到物理地址的重映射，从而提高系统的性能
    * 原先：通常情况下，如果两个进程占用的虚拟地址空间有重叠，系统在这两个进程之间进行切换时，必须进行虚拟地址到物理地址的重映射。而虚拟地址到物理地址的重映射涉及到**重建MMU中的页表**，而且**cache及TLB中的内容都必须使无效**（通过设置协处理器寄存器的相关位）。这些操作将带类巨大的系统开销，一方面重建MMU和使无效cache及TLB的内容需要很大的开销，另一方面重建cache和TLB内容也需要很大的开销。
    * 改进：快速上下文切换机构将个进程的虚拟地址空间变换成不同的虚拟地址空间。这样在进行进程间切换时就不需要进行虚拟地址到物理地址的重映射。

## cortex-a series

### performance

* [performance of all arm cortex series](https://en.wikipedia.org/wiki/List_of_ARM_microarchitectures) (也可参考[ARM Cortex-A](https://en.wikipedia.org/wiki/ARM_Cortex-A))
    * 指标：
        * DMIPS:Dhrystone Million Instructions executed Per Second ：主要用于测整数计算能力。D是Dhrystone的缩写，他表示了在Dhrystone这样一种测试方法下的MIPS
        * MFLOPS:Million Floating-point Operations per Second：主要用于测浮点计算能力。
    * 数据

        |type|arch|perf(DMIPS/MHz)|
        |---|---|---|
        |Cortex A5|v7|1.57|
        |Cortex A7|v7|1.9|
        |Cortex A8|v7|2.0|
        |Cortex A9|v7|2.5|
        |Cortex A12|v7|3.0|
        |Cortex A15|v7|>3.5|
        |Cortex A17|v7|2.8|
        |Cortex A35|v8|1.78|
        |Cortex A53|v8|2.3|
        |Cortex A57|v8|4.1|
        |Cortex A72|v8|4.7|
        |Cortex A73|v8|4.9|

* performance of old cortex v7

    ![cortex-a-perf.jpg](./data/soc/cortex-a-perf.jpg)
    * refer: [Floating-point performance of ARM cores and their efficiency in classical molecular dynamics](https://iopscience.iop.org/article/10.1088/1742-6596/681/1/012049/pdf)
* performance of cortex-M(computing int8/float multiply operation)
    * cortex m3: cycles for each FLO in compiler supported platform(cortex M3, compile implement the operation) (**The integer multiplication took 7 cycles**, of which **4 cycles** were used to **load** operands and **2 cycles** - to **store** the result. The **multiplication itself is 1 cycle**, in accordance with 'hardware single-cycle multiply' promise of Cortex-M3.**The float multiplication took 47 cycles** with the **multiplication itself taking 41 cycles**. Keep in mind that the float multiplication execution time depends on the values of operands. refer: [Floating point performance on Cortex M3](https://community.arm.com/developer/tools-software/tools/f/keil-forum/28234/floating-point-performance-on-cortex-m3)

## ABI

<span id="arm-abi"></span>

### 标准
* APCS([arm procedure call standard](https://www.cl.cam.ac.uk/~fms27/teaching/2001-02/arm-project/02-sort/apcs.txt))
* [ARM 架构参考手册](https://www.scss.tcd.ie/~waldroj/3d1/arm_arm.pdf)
* [ARM 架构的过程调用标准](https://static.docs.arm.com/ihi0042/i/aapcs32.pdf?_ga=2.46926119.1229532213.1584787764-997854972.1583052689)(Procedure Call Standard for the Arm Architecture)
* [ARM ELF 文件格式](http://infocenter.arm.com/help/topic/com.arm.doc.dui0101a/DUI0101A_Elf.pdf)
* [针对 ARM 体系结构的应用程序二进制接口 (ABI)](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.subset.swdev.abi/index.html)

### 应用范式

* [stack operations for nested subroutines](http://infocenter.arm.com/help/topic/com.arm.doc.dui0473j/dom1359731152874.html)
    * lr和工作寄存器均要保存
    ```asm
    subroutine PUSH {r5-r7,lr} ; Push work registers and lr
    ; code
    BL somewhere_else
    ; code
    POP {r5-r7,pc} ; Pop work registers and pc
    ```

## instruction

### 指令集与内嵌汇编

* thumb & arm指令集
    * thumb指令集可以认为是16位精简版的arm指令集，提供通用功能，但在必要的情况下依然需要使用arm指令集完成操作，比如异常等
    * 设置gcc编译指令集：**默认编译为arm指令集**，在android中，给gcc传入了-mthumb参数后，尽可能**编译成thumb指令集压缩空间**
    * 参考：[Differences between Thumb and ARM instruction sets](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0068b/ch02s02s09.html)
* inline assembly
    * 通用格式
    
        ```asm
        "cmd1\n"
        "cmd2\n"
        ...
        : [output]  "format" (c var)
        : [input] "format" (c var)
        : extra constraints
        ```

    * 示例
        ```asm
        asm volatile(
        "mov r0, %1\n"
        "mov r1, %2\n"
        "mov r2, %3\n"
        "mov r3, %4\n"
        "ldr r4, =syscallX\n"
        "mov %0, r4\n"
        "bl syscallX\n"
        : "=r" (cpy)
        : "r" (num), "r" (dir), "r" (mode), "r"(ptr)
        : "r0", "r1", "r2", "r3", "r4", "memory"
        );
        ```
    * 其中
        * a) `memory` is a valid keyword too. It tells the compiler that the assembler instruction may change memory locations.This forces the compiler to store all cached values before and reload them after executing the assembler instructions.
        * b) `[output]  "format" (c var)` 格式可以简化为 `"format (c var)"`， 同时在assembly中用`%`数字代表参数，从0开始，比如`%0`代表第一个输出
        * c) 在output和Input格式中 `"="` Write-only operand, usually used for all output operands `"+"` Read-write operand, must be listed as an output operand
        * d)常用的无用代码比如 mov r0, r0
    * 参考：[ARM GCC Inline Assembler Cookbook](http://www.ethernut.de/en/documents/arm-inline-asm.html)，[Assembler Instructions with C Expression Operands](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Volatile)

### 基本指令

* **跳转**([B, BL, BX, BLX, and BXJ](https://developer.arm.com/docs/dui0489/h/arm-and-thumb-instructions/b-bl-bx-blx-and-bxj)])
    * `bx {<cond>} <Rm>`
        * \<cond\>为指令执行的条件码。当\<cond\>忽略时指令为无条件执行。
        * \<Rm\>该寄存器中为跳转的目标地址。当\<Rm\>寄存器的bit[0]为0时，目标地址处的指令为ARM指令；当\<Rm\>寄存器的bit[0]为1时，目标地址处的指令为Thumb指令。
    * `bl {cond} label`
        * 其作用是，除了b指令跳转到label之外，在跳转之前，先把下一条指令地址存到lr寄存器中，以方便跳转到那边执行完毕后，将lr再赋值给pc，以实现函数返回，继续执行下面的指令的效果。

            ```
                bl    cpu_init_crit
            ......
            cpu_init_crit:
            ......
                mov    pc, lr
            ```
            
            对应c代码为

            ```c
            cpu_init_crit();
            ......
            void cpu_init_crit(void)
            {
                ......
            }
            ```
    * 核心差异
        * The BL and BLX instructions copy the address of the next instruction into lr (r14, the link register).
    	* The BX and BLX instructions can change the processor state from ARM to Thumb, or from Thumb to ARM.
<span id="ldr-str"></span>

* **读写**(mov & ldr & str & stm & ldm & strex & ldrex)
    * ldr指令数据从内存中某处读取到寄存器中，ldr伪指令可以在立即数前加上=，以表示把一个地址写到某寄存器中
    * `mov`只能在寄存器之间移动数据，或者把立即数移动到寄存器中
    * `ldr`伪指令和mov是比较相似的，只是mov指令限制了立即数的长度为8位
    * `str`指令用从源寄存器中将一个32位的字数据传送到存储器中
        * `str {cond}  src_reg, <ddr_addr>`
        * `str r0, [r1], #8`；将R0中的字数据写入以R1为地址的存储器中，并将新地址R1＋8写入R1
        * `str r0, [r1, #8]`；将R0中的字数据写入以R1＋8为地址的存储器中
        * `str r1, [r0]`；将r1寄存器的值，传送到地址值为r0的（存储器）内存中
    * `ldm`是`ldr`的扩展版，用于load，这个指令运行的方向和LDR是不一样的，是从左到右运行的，该指令是将内存中堆栈内的数据，批量的赋值给寄存器。
    * `stm`是`str`的扩展版，用于store，区别于STR，是将堆栈指针写在左边，而把寄存器组写在右边。
    * 独占访问指令：
        * `ldrex`：`LDREX Rx, [Ry]`读取寄存器Ry指向的4字节内存值，将其保存到Rx寄存器中，同时标记对Ry指向内存区域的独占访问。
        * `strex`：`STREX Rx, Ry, [Rz]`如果执行这条指令的时候发现已经被标记为独占访问了，则将寄存器Ry中的值更新到寄存器Rz指向的内存，并将寄存器Rx设置成0。指令执行成功后，会将独占访问标记位清除。而如果执行这条指令的时候发现没有设置独占标记，则不会更新内存，且将寄存器Rx的值设置成1。一旦某条STREX指令执行成功后，以后再对同一段内存尝试使用STREX指令更新的时候，会发现独占标记已经被清空了，就不能再更新了，从而实现独占访问的机制。
        * 参考：[ARM平台下独占访问指令LDREX和STREX的原理与使用详解](https://blog.csdn.net/adaptiver/article/details/72392825)；[arm exclusive access](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dht0008a/CJAGCFAF.html&_ga=2.172681764.640980534.1584281365-997854972.1583052689)；[基于strex/ldrex的pthread实现](./linux_syscall.md#mutex-syscall)
* **控制**([fp & sp & lr & pc](https://stackoverflow.com/questions/15752188/arm-link-register-and-frame-pointer))
    * The pc and lr are related. One is "**where you are**" and the other is "**where you were**".
    * sp is where the **stack is** and the fp is where the **stack was**
    * a stack frame layout
        * fp[-0] saved pc, where we stored this frame.
        * fp[-1] saved lr, the return address for this function.
        * fp[-2] previous sp, before this function eats stack.
        * fp[-3] previous fp, the last stack frame.
* msr/mrs
    * CPSR/SPSR寄存器保存了用户模式和系统模式等状态信息，对CPSR，SPSR寄存器进行操作不能使用mov，ldr等通用指令，只能使用特权指令msr和mrs
    * MRS（Move to Register from State register）
    * MSR（Move to State register from register）
    * 小部分位段可以在user space更改，所有位段都能在kernel space更改，指令集mode段只能在kernel space更改
* swi/svc
    * swi和svc均为系统调用
    * SWI and SVC are same thing, it is just a name change. Previously, the SVC instruction was called SWI, Software Interrupt.
* dmb/dsb/isb
    * 存在价值
        * 解决arm做指令乱序时产生的load/store乱序导致的影响；指令乱序不会对存在数据依赖的代码进行乱序（但有些数据依赖，比如并行或者driver应用，是无法通过代码字面情形分析出来的）
    * 受众
        * 应用程序开发人员,无须过多担心它，使用的任何并发框架都可以为您解决它；
        * 设备驱动程序开发人员,那么您将很容易找到示例-只要您的代码在执行某些其他访问之前依赖于先前的访问产生了影响(清除了中断源,编写了DMA描述符), (重新启用中断,启动DMA事务)；
        * 开发并发框架(或调试其中一个框架),则可能需要多阅读一些有关该主题的信息
    * 区别
        * DMB：数据存储器隔离。DMB 指令保证： 仅当所有在它前面的存储器访问操作都执行完毕后，才提交(commit)在它后面的存储器访问操作。
        * DSB:数据同步隔离。比 DMB 严格： 仅当所有在它前面的存储器访问操作都执行完毕后，才执行在它后面的指令
        * ISB：指令同步隔离。最严格：它会清洗流水线，以保证所有它前面的指令都执行完毕之后，才执行它后面的指令。
    * 其他
        * compiler barrier
            * volatile：让编译器生成的代码，每次都从内存重新读取变量的值，而不是用寄存器中暂存的值。
            * linux barrier()：`#define barrier() __asm__ __volatile__("": : :"memory")`
        * memory barrier
            * dmb/dsb/isb in arm
    * 参考
        * [对优化说不 - Linux中的Barrier](https://zhuanlan.zhihu.com/p/96001570)
        * [Memory access ordering - an introduction](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/memory-access-ordering---an-introduction)
        * [Memory access ordering part 2: Barriers and the Linux kernel](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/memory-access-ordering-part-2---barriers-and-the-linux-kernel)
        * [Memory access ordering part 3: Memory access ordering in the Arm Architecture](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/memory-access-ordering-part-3---memory-access-ordering-in-the-arm-architecture)
        * [Memory Barriers: a Hardware View for Software Hackers](http://www.rdrop.com/users/paulmck/scalability/paper/whymb.2010.07.23a.pdf)
* teq
    * test equal，如果两者相等将，CPSR状态寄存器的EQ status置位，当遇到bne等指令时，如果equal，则执行，否则放弃。


# ASIC(for AI)

* 寒武纪DianNao系列NPU设计
    * [DianNao: A Small-Footprint High-Throughput Accelerator for Ubiquitous Machine-Learning](http://novel.ict.ac.cn/ychen/pdf/DianNao.pdf) 2014
    * [DaDianNao: A Machine-Learning Supercomputer](http://novel.ict.ac.cn/ychen/pdf/DaDianNao.pdf) 2014 - for server end
    * [ShiDianNao: Shifting Vision Processing Closer to the Sensor](https://www.epfl.ch/labs/lap/wp-content/uploads/2018/05/DuJun15_ShiDianNaoShiftingVisionProcessingCloserToTheSensor_ISCA15.pdf) 2015 - for embedding end
    * [PuDianNao: A Polyvalent Machine Learning Accelerator](https://dl.acm.org/doi/10.1145/2694344.2694358) 2015
    * [中科院说的深度学习指令集diannaoyu到底是什么?](https://www.zhihu.com/question/41216802)
* NPU加速设计思想
    * NFU和片上存储的时分复用特性。针对一个大网络，其模型参数会依次被加载到SB里，每层神经layer的输入数据也会被依次加载到NBin，layer计算结果写入到NBout。
    * Adder/Multiplier/AdderTree提供对全连接操作的快速支持
    * 加速芯片SRAM与传感器直连，减少两次DRAM的数据搬移（合理设计）
    * 合理利用AI计算场景中的data locality
    * 在服务器端芯片中，使用eDRAM代替SRAM/DRAM，在存储密度/访存延迟/功耗之间获得了大模型所需的更适宜的trade-off（服务器场景中模型更大，计算能耗更高）

    ![diannao](./data/soc/diannao.png)

    ![dadiannao](./data/soc/dadiannao.png)