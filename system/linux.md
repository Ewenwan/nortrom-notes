
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

## 日志

* 日志系统
    * 内核日志
        * **printk**: The kernel logs messages (using the `printk()` function) to a ring buffer in kernel space. These messages are made available to user-space applications in two ways: via the `/proc/kmsg` file (provided that /proc is mounted), and via the `sys_syslog` syscall.
        * **get printk msg**: There are two main applications that read (and, to some extent, can control) the kernel's ring buffer: `dmesg(1)` and `klogd(8)`
    * 运行时日志
        * **syslog**: User-space applications normally use the libc function `syslog(3)` to log messages. libc sends these messages to the UNIX domain socket `/dev/log`
    * 参考：[Understand logging in Linux](https://unix.stackexchange.com/questions/205883/understand-logging-in-linux)

# 系统调用

* [各平台系统调用指令](http://man7.org/linux/man-pages/man2/syscall.2.html)
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
<span id="mutex-syscall"></span>

* semop & [futex](http://man7.org/linux/man-pages/man2/futex.2.html)
    * 锁机制概述
        * Mutex provide mutual exclusion by having a memory area which can toggle between locked and unlocked atomically. The memory resides in user space but the toggling happens through a system call inside the kernel. The reason why the system call is required (and hence heavy weight) is because:
            * WAKE - The kernel has to schedule the next thread in the wait queue
            * WAIT - All accesses to the lock are guarded and queued through a single point of synchronization - the kernel.
    * semop
        * 传统的SystemV IPC(inter process communication)进程间同步机制都是通过内核对象来实现的，以 semaphore 为例，当进程间要同步的时候，必须通过系统调用semop(2)进入内核进行PV操作。系统调用的缺点是开销很大，需要从user mode切换到kernel mode、保存寄存器状态、从user stack切换到kernel stack、等等，通常要消耗上百条指令。
    * futex
        * futex (fast userspace mutex) 是Linux的一个基础组件，可以用来构建各种更高级别的同步机制，比如锁或者信号量等等，POSIX信号量就是基于futex构建的。大多数时候编写应用程序并不需要直接使用futex，一般用基于它所实现的系统库就够了。
        * futex的解决思路是：**在无竞争的情况下操作完全在user space进行，不需要系统调用，仅在发生竞争的时候进入内核去完成相应的处理(wait 或者 wake up)**。所以说，futex是一种user mode和kernel mode混合的同步机制，需要两种模式合作才能完成，futex变量必须位于user space，而不是内核对象，futex的代码也分为user mode和kernel mode两部分，无竞争的情况下在user mode，发生竞争时则通过sys_futex系统调用进入kernel mode进行处理
        * Futex isn't the same as mutex. It provides **a wait queue technique accessible in the user space**. **Pthread Mutex** in the latest Linux distributions are implemented through Futex logic.
        * futex代码示例
        ```c
        pthread_mutex_lock
            if mutex type is MUTEX_TYPE_NORMAL -> _normal_lock
                if (__atomic_cmpxchg(shared|0, shared|1, &mutex->value ) != 0) {  -->如果value不为0(unlock)，进入下一步，如果value为0，把value设置为1（lock）
                while (__atomic_swap(shared|2, &mutex->value ) != (shared|0))  -->将value设置为2（contend），返回值为过去的value，如果不为0继续睡，注意当被唤醒后value依旧被设置为2
                        __futex_wait_ex(&mutex->value, shared, shared|2, 0);   -->等待被唤醒，如果value为2(contend)，则继续睡
            ANDROID_MEMBAR_FULL(); --> do { __asm__ __volatile__ ("mfence" ::: "memory"); } while (0) sync和reload所有的cache
        ```
            
        ```c
        pthread_mutex_unlock
        ANDROID_MEMBAR_FULL
        if (__atomic_dec(&mutex->value) != (shared|1)) {  -->对value进行减1操作，并将过去的value值与1（lock）比较，如果存在竞态，需要进一步唤醒在sleep的竞争者
            mutex->value = shared;  -->设置value为0, 与lock的while循环判断条件对应
            __futex_wake_ex(&mutex->value, shared, 1);  -->唤醒
        ```

        ```c
        pthread_cond_wait
        pthread_mutex_unlock(mutex);
        status = __futex_wait_ex(&cond->value, COND_IS_SHARED(cond), oldvalue, reltime);
        pthread_mutex_lock(mutex);
        由于futex可能会被误唤醒，所以唤醒后需要添加判断条件以确保被正确唤醒
        pthread_mutex_lock
        while(cond not come)  pthread_cond_wait
        pthread_mutex_unlock
        ```

        * `__atomic_cmpxchg`在arm中是通过`ldrex`和`strex`独占指令实现的，可参见[arm读写指令](./soc.md#ldr-str)，具体实现可参见src：[pthread.c](https://android.googlesource.com/platform/bionic/+/10ce969/libc/bionic/pthread.c),[bionic_atomic_arm.h](https://android.googlesource.com/platform/bionic/+/e31bfae2baa96742f998155ee26e56c826a8ce3a/libc/private/bionic_atomic_arm.h)
        * 总的来说，当前大多数用户态的锁都是通过futex系统调用实现的，目前的分工是：**用户态负责维护lock state**(like the value of futex is waiting, 0 for no contend, 1 for locked, 2 for contended)；而**内核则只负责维护process waiter queue**
        * 此外，用户态在实现锁操作时，在wait and check中一般会有retry机制，也就是上文中的：这种机制保证了checkpoint时虽然只保存了用户态锁状态，但当restart时，thread在从signal handler返回到kernel后，会立马返回到user space携带EINTR,且等待条件不满足，因而继续wait，调用futex，内核态waiter queue得以建立。

        ```c
        while (__atomic_swap(shared|2, &mutex->value ) != (shared|0)) -->将value设置为2（contend），返回值为过去的value，如果不为0继续睡，注意当被唤醒后value依旧被设置为2
            __futex_wait_ex(&mutex->value, shared, shared|2, 0); -->等待被唤醒，如果value为2(contend)，则继续睡
        ```

    * 参考：[How different is a futex from mutex - conceptually and also implementation wise?](https://www.quora.com/How-different-is-a-futex-from-mutex-conceptually-and-also-implementation-wise)
    * 参考：futex在android pthread中的应用，[同步应用](#sync-application)
* exec
    * execve为内核级系统调用，其他（execl，execle，execlp，execv，execvp）都是调用execve的库函数
    * exec系列中，**e代表指定环境变量**，**v指代参数list**,**p指代PATH**
        * The `execv()`, `execvp()`, and `execvpe()` functions provide an array of pointers to **null-terminated strings that represent the argument list** available to the new program.
            * The first argument, by convention, should point to the filename associated with the file being executed. The array of pointers must be terminated by a NULL pointer.
        * The `execle()` and `execvpe()` functions allow the caller to **specify the environment** of the executed program via the argumentenvp.
        * The `execlp()`, `execvp()`, and `execvpe()` functions duplicate the actions of the shell in searching for an executable file if the specified filename does not contain a slash (/) character. The file is sought in the colon-separated list of directory pathnames specified **in the PATH environment variable**.
* [chrt](http://man7.org/linux/man-pages/man1/chrt.1.html)
    * chrt change the real-time attributes of a process(priority, schedule type, etc)
* [setuid](http://man7.org/linux/man-pages/man2/setuid.2.html)
    * uid/gid：真实的用户、组（uid、gid）：进程的真正所有者。每当用户在shell终端登录时，都会将登录用户作为登录进程的真正所有者。
    * euid/egid：有效的用户、组（euid、egid）：进程的有效用户、组。进程所执行各种操作所允许的权限（process credentials）是依据进程的有效用户来判断的
    * situation of euid=0(root)
        * `setuid` will permanently changes the uid
        * `seteuid` temporarily changes uid and then use `setuid(0)` to change it back
    * situation of euid>0(not root)
        * `setuid/seteuid` changes its effective uid to only its original euid uid or suid
* sigpending
    * sigpending() user space has interface of pending signals and record it
    * returns the set of signals that are pending for delivery to the calling thread
* prctl
    * get process name
    * prctl() is called with a first argument describing what to do (with values defined in \<linux/prctl.h\>), and further arguments with a significance depending on the first one
* [ptrace](http://man7.org/linux/man-pages/man2/ptrace.2.html)
    * The `ptrace()` system call provides a means by which one process (the "tracer") may observe and control the execution of another process (the "tracee"), and examine and change the tracee's memory and registers.  It is primarily used to implement breakpoint debugging and system call tracing.
    * strace就是使用了ptrace系统调用实现的工具，可使用ptrace实现一个简易的strace工具，参考[Write yourself an strace in 70 lines of code](https://blog.nelhage.com/2010/08/write-yourself-an-strace-in-70-lines-of-code/)
    
    ```c
    int main(int argc, char **argv) {
        if (argc < 2) {
            fprintf(stderr, "Usage: %s prog args\n", argv[0]);
            exit(1);
        }

        pid_t child = fork();
        if (child == 0) {
            return do_child(argc-1, argv+1);
        } else {
            return do_trace(child);
        }
    }

    int do_child(int argc, char **argv) {
        char *args [argc+1];
        memcpy(args, argv, argc * sizeof(char*));
        args[argc] = NULL;

        ptrace(PTRACE_TRACEME);
        kill(getpid(), SIGSTOP);
        return execvp(args[0], args);
    }

    int wait_for_syscall(pid_t child);

    int do_trace(pid_t child) {
        int status, syscall, retval;
        waitpid(child, &status, 0);
        ptrace(PTRACE_SETOPTIONS, child, 0, PTRACE_O_TRACESYSGOOD);
        while(1) {
            if (wait_for_syscall(child) != 0) break;

            syscall = ptrace(PTRACE_PEEKUSER, child, sizeof(long)*ORIG_EAX);
            fprintf(stderr, "syscall(%d) = ", syscall);

            if (wait_for_syscall(child) != 0) break;

            retval = ptrace(PTRACE_PEEKUSER, child, sizeof(long)*EAX);
            fprintf(stderr, "%d\n", retval);
        }
        return 0;
    }

    int wait_for_syscall(pid_t child) {
        int status;
        while (1) {
            ptrace(PTRACE_SYSCALL, child, 0, 0);
            waitpid(child, &status, 0);
            if (WIFSTOPPED(status) && WSTOPSIG(status) & 0x80)
                return 0;
            if (WIFEXITED(status))
                return 1;
        }
    }
    ```

* [waitpid](http://linux.die.net/man/2/waitpid)
    * All of these system calls are used to wait for state changes in a child of the calling process, and obtain information about the child whose state has changed. A state change is considered to be: the child terminated; **the child was stopped by a signal; or the child was resumed by a signal**. In the case of a terminated child, performing a wait allows the system to release the resources associated with the child; **if a wait is not performed, then the terminated child remains in a "zombie" state** (see NOTES below).
    * options(wait意味着wait所有的子孙且处于阻塞状态)
        * < -1 meaning wait for any child process whose process group ID is equal to the absolute value of pid.
        * -1 meaning wait for any child process.
        * 0 meaning wait for any child process whose process group ID is equal to that of the calling process.
        * \> 0 meaning wait for the child whose process ID is equal to the value of pid.
* [sigaction](http://man7.org/linux/man-pages/man2/sigaction.2.html)
    * The sigaction() system call is used to change the action taken by a process on receipt of a specific signal.
    
    ```c
    struct sigaction act;
    memset(&act, 0, sizeof(act));
    act.sa_handler = sigchld_handler;
    act.sa_flags = SA_NOCLDSTOP;
    sigaction(SIGCHLD, &act, 0);
    ```
* socketpair
    * socketpair创建了一对无名的套接字描述符（只能在AF_UNIX域中使用），描述符存储于一个二元数组,eg. s[2] .这对套接字可以进行双工通信，每一个描述符既可以读也可以写。 这和pipe有一定的区别，pipe是单工通信，一端要么是读端要么是写端，而socketpair实现了双工套接字，也就没有所谓的读端和写端的区分
* [fcntl](http://man7.org/linux/man-pages/man2/fcntl.2.html)
    * performs one of the operations described below on the open file descriptor fd. The operation is determined by cmd. 
        * F_SETFD Set the file descriptor flags to the value specified by arg. 
        * F_SETFL Set the file status flags to the value specified by arg. File access mode
            * `fcntl(fd, F_SETFD, FD_CLOEXEC) == -1)` FD_CLOEXEC It marks the file descriptor so that it will be close()d automatically when the process or any children it fork()s calls one of the exec*() family of functions. This is useful to keep from leaking your file descriptors to random programs run by e.g. system(). 参考：[What does the FD_CLOEXEC fcntl() flag do?](https://stackoverflow.com/questions/6125068/what-does-the-fd-cloexec-fcntl-flag-do)
* [inotify](http://man7.org/linux/man-pages/man7/inotify.7.html)
    * The inotify API provides a mechanism for monitoring filesystem events.  Inotify can be used to monitor individual files, or to monitor directories.  When a directory is monitored, inotify will return events for the directory itself, and for files inside the directory.


# 设备

## udev

* **what is udev**: It provides a dynamic device directory containing only the files for actually present devices.
    * It creates or removes device node files usually located in the /dev directory, or it renames network interfaces.
    * a udev related file path include:
        * /sbin/udev udev program
        * /etc/udev/* udev config files
        * /etc/hotplug.d/default/udev.hotplug hotplug symlink to udev program
        * /etc/dev.d/* programs invoked by udevudev to automount -- step after udev differs in many ways
* **the general flow** is: kernel -> udev -> dbus -> hal -> gnome-vfs/nautilus (mount) with these work being done:
    * It will automatically mount USB drives on plugin, and shouldn't take much to adapt for Firewire.
    * It uses UDEV, so no monkeying with HAL/DeviceKit/GNOME-Anything.
    * It automatically creates a /media/LABEL directory to mount the device to.
    * 参考：[Automatically mount external drives to /media/LABEL on boot without a user logged in?](https://superuser.com/questions/53978/automatically-mount-external-drives-to-media-label-on-boot-without-a-user-logge)
* **command** use `lsusb` & `dmesg` to check current device info like

# 机制剖析

# 应用

## 同步
<span id="sync-application"></span>

* pthread 同步访问同一资源
    * thread-1

    ```c
    pthread_mutex_lock(&mutex);
    // 设置条件为true
    ... 
    pthread_cond_signal(&cond);  // 条件满足时通知线程二
    pthread_mutex_unlock(&mutex);
    ```

    * thread-2
    
    ```c
    pthread_mutex_lock(&mutex);
    while (条件为false) {
        pthread_cond_wait(&cond, &mutex); // 永久或超时等待线程一通知条件满足
        ... 
    }
    pthread_mutex_unlock(&mutex);
    ```

    * pthread_cond_wait 条件变量
        * pthread_cond_wait()函数一进入wait状态就会自动release mutex.
        * pthread_cond_wait() 一旦wait成功获得cond 条件的时候会自动 lock mutex.
        * 条件变量是用来等待而不是用来上锁的。条件变量用来自动阻塞一个线程，直到某特殊情况发生为止。通常条件变量和互斥锁同时使用。
    * 备注：
        * 1) 第二段代码之所以在pthread_cond_wait外面包含一个while循环不停测试条件是否成立的原因是, 在pthread_cond_wait被唤醒的时候可能该条件已经不成立.参考：[Calling pthread_cond_signal without locking mutex](https://stackoverflow.com/questions/4544234/calling-pthread-cond-signal-without-locking-mutex)
        * 2) pthread_cond_wait调用必须和某一个mutex一起调用, 这个mutex是在外部进行加锁的mutex, 在调用pthread_cond_wait时, 内部的实现将首先将这个mutex解锁, 然后等待条件变量被唤醒, 如果没有被唤醒, 该线程将一直休眠, 也就是说, 该线程将一直阻塞在这个pthread_cond_wait调用中, 而当此线程被唤醒时, 将自动将这个mutex加锁.
    * 参考：[互斥量、条件变量与pthread_cond_wait()函数的使用](https://blog.csdn.net/cnclenovo/article/details/44589221)
* 多进程读写同一文件
    * 概念
        * **两个线程同时write一个文件是不会冲突**。所有的系统调用都是原子性的：All system calls are executed atomically. By this, we mean that the kernel guarantees that all of the steps in a system call are completed as a single operation, without being interrupted by another process of thread。
        * **多个系统调用写文件会冲突，使用文件的O_APPEND解决**。比如以下代码：
        
        ```c
        lseek(fd,0,SEEK_END); //seek to the end of the file
        write(fd,"log message",len); //perform the write
        ```

        * **多个库函数写文件存在冲突**。
            * 如果同时向同一个文件的同一片区域写入，势必出问题，一来写的结果可能被覆盖，二来写操作可能交替进行导致结果不可辨识。
            * 如果同时向同一个文件的不同区域写入，直接使用syscall，不会有问题，因为系统只有该文件的一份cache。
            * 如果同时向同一个文件的不同区域写入，使用运行时库或者其他中间平台提供的api，则结果不可预测，因为可能中间会有额外的cache，cache不一致的直接后果就是结果不可知。
    * 如何加锁
    * **advisory lock -- 若想对文件加锁访问，则使用flock和fcntl**。`flock()`：文件级别的锁，针对整个文件进行加锁。`fcntl()`函数：段级别的锁，能够针对文件的某个部分进行加锁。
    * **mandatory lock**.mandatory file lock need to mount the filesystem with -o option and set special mode of the file you want to lock
        * `mount -oremount,mand / chmod g+s,g-x mandatory.txt`
    * **最复杂完美的方式为类似于Linux日志文件服务**。启动一个logger进程，其他进程向logger发消息，即把数据发送给logger，由logger来写文件，这种方法最安全，但是实现上相对复杂
    * 参考
        * [Is lock-free logging safe?](https://www.jstorimer.com/blogs/workingwithcode/7982047-is-lock-free-logging-safe)
        * [Linux 多进程读写文件 文件锁](http://blog.chinaunix.net/uid-11572501-id-4126870.html)
        * [2 Types of Linux File Locking](https://www.thegeekstuff.com/2012/04/linux-file-locking-types/)
* 各类同步机制区别
    * mutex: 调用进程可以sleep 一个进程进入临界区
    * semaphore: 调用进程可以sleep 多个进程可进入临界区（主要的应用是共享内存方式的进程间通信）
    * spin_lock: 调用进程不可以sleep 针对小资源 一个进程进入临界区
    * 三者在内核中均基于spinlock实现
    * 应用差异
        * **使用cond时，线程间存在依赖关系**。**只使用mutex时，线程间为对等关系**。
        * cond适用于event或者说是edge triggered event或者切换状态，是多个使用者均在等待一个即将出现的资源
        * lock则使用与已有资源，计数等，可以理解为level triggered event，是多个使用者轮番使用某个已有资源。
    * 参考
        * [信号量、互斥体和自旋锁](https://www.cnblogs.com/biyeymyhjob/archive/2012/07/21/2602015.html)
        * [pthread_cond_wait and mutex requirement](https://stackoverflow.com/questions/6312342/pthread-cond-wait-and-mutex-requirement)
* 可重入&线程安全
    * 可重入
        * 定义：可以被中断；除了使用自己栈上的变量以外不依赖于任何环境（包括static）；可以允许有多个该函数的副本在运行
        * 应用：Linux 中可重入这个概念一般只有在 signal 的场景下有意义，叫 async-signal-safe。
    * 区别
        * async-signal safe is stronger because **two invocations of the function can be in the same thread**.
        * 很多线程安全的函数都是不可重入的，例如 malloc,printf。因此，一个函数不可重入是因为1.使用了全局或静态变量2.调用了malloc/free3.调用了标准I/O函数
        * 可重入的函数一般也是线程安全的，反例：（signal中包含和thread同样的Lock，参见：[异步可重入函数与线程安全函数等价吗？](https://www.zhihu.com/question/21526405)）。
        * Posix中大多数函数都是线程安全的，但只有少数是 async-signal-safe。
    * 参考
        * [C Thread Safe and Reentrant Function Examples](https://www.thegeekstuff.com/2012/07/c-thread-safe-and-reentrant/)
        * [What exactly is a reentrant function?](https://stackoverflow.com/questions/2799023/what-exactly-is-a-reentrant-function)
* 各平台线程复位机制
    * 条件变量的置位和复位有两种常用模型：第一种模型是当条件变量置位（signaled）以后，如果当前没有线程在等待，其状态会保持为置位（signaled），直到有等待的线程进入被触发，其状态才会变为复位（unsignaled），这种模型的采用以 Windows 平台上的 Auto-set Event 为代表。其状态变化如图所示：

    ![window_event.gif](./data/linux/window_event.gif)

    * 第二种模型则是 Linux 平台的 Pthread 所采用的模型，当条件变量置位（signaled）以后，即使当前没有任何线程在等待，其状态也会恢复为复位（unsignaled）状态。其状态变化如图所示：具体来说，Linux 平台上 Pthread 下的条件变量状态变化模型是这样工作的：调用 pthread_cond_signal() 释放被条件阻塞的线程时，无论存不存在被阻塞的线程，条件都将被重新复位，下一个被条件阻塞的线程将不受影响。而对于 Windows，当调用 SetEvent 触发 Auto-reset 的 Event 条件时，如果没有被条件阻塞的线程，那么条件将维持在触发状态，直到有新的线程被条件阻塞并被释放为止。

    ![linux_pthread.gif](./data/linux/linux_pthread.gif)

    * 参考：[Linux 的多线程编程的高效开发经验](https://www.ibm.com/developerworks/cn/linux/l-cn-mthreadps/)
* 其他链接
    * [futex](#mutex-syscall)