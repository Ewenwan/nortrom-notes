
[toc]

[android](./android.md)

# zram原理

zRAM可以分出一块内存，然后让系统当作虚拟内存来使用。传统的虚拟内存是存放在磁盘上的，而zRAM存在内存里，并会进行压缩。这样的虚拟内存访问速度可以提高很多

# make zram works

* 配置kernel config
    * `make kmenuconfig`查找并enable如下配置
    ```
    CONFIG_ZRAM=y
    CONFIG_ZSMALLOC=y
    CONFIG_SWAP=y
    CONFIG_ZCACHE=y
    CONFIG_ZCACHE_DEBUG=y
    CONFIG_CLEANCACHE=y
    CONFIG_FRONTSWAP=y
    ```
* 详见：[zram-android](https://source.android.com/devices/tech/perf/low-ram#zram), [set up config](https://github.com/raspberrypi/linux/issues/179)

# 手工设置并swapon zram分区

* 方式
    ```
    echo $((150*1024*1024)) > /sys/block/zram0/disksize
    echo $((50*1024*1024)) > /sys/block/zram0/disksize
    mkswap /dev/block/zram0
    swapon -p 10 /dev/block/zram0
    ```
* 参考
    * [Compressed RAM based block devices](https://www.kernel.org/doc/Documentation/blockdev/zram.txt)

# 如何查看是否enable

* 使用命令
    * `free && cat /proc/swaps` 查看总的swap使用情况
    * `cat /proc/2156/smaps | grep Swap | grep -v 0` 查看某pid的swap情况（system/bin/）该目录下的daemon等进程普遍使用了swap

# 测试zram性能

* 将file放在swap中，测试其与正常状态下的读写性能
    * [zramperf](https://code.google.com/p/compcache/wiki/zramperf)
    * [android-zram-status](http://adhisimon.github.io/android-zram-status/)
* 分析内存和swap

    ```
    echo -e "shared_dirty\tshared_clean\tprivate_dirty\tprivate_clean\tproc_name"

    for pid in `ls -l /proc | grep ^d | awk '{print $9}' | grep [0-9]`
    do
        if [ -e "/proc/$pid/smaps" ]
        then
            shared_dirty=$(grep Shared_Dirty /proc/$pid/smaps | awk '{ sum+=$2;} END{ print sum }')
            shared_clean=$(grep Shared_Clean /proc/$pid/smaps | awk '{ sum+=$2;} END{ print sum }')
            private_dirty=$(grep Private_Dirty /proc/$pid/smaps | awk '{ sum+=$2;} END{ print sum }')
            private_clean=$(grep Private_Clean /proc/$pid/smaps | awk '{ sum+=$2;} END{ print sum }')
            if [ -z "$shared_dirty" ]
            then
                    shared_dirty="0"
            fi
            if [ -z "$shared_clean" ]
            then
                    shared_clean="0"
            fi
            if [ -z "$private_dirty" ]
            then
                    private_dirty="0"
            fi
            if [ -z "$private_clean" ]
            then
                    private_clean="0"
            fi
            proc_name=$(ps | grep -w "$pid" | grep -v grep | awk '{print $5}')
            echo -e "$shared_dirty\t\t$shared_clean\t\t$private_dirty\t\t$private_clean\t\t$proc_name"
        fi
    done
    ```

