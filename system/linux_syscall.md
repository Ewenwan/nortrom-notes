
[toc]

[linux](./linux.md)


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
    * 参考代码：
        ```
        ~/repo_44/android/bionic/libc/kernel/arch-arm/asm/user.h
        ~/repo_44/android/bionic/libc/kernel/arch-arm/asm/ptrace.h
        ~/repo_44/android/bionic/libc/arch-arm/bionic/syscall.S
        ~/repo_44/android/bionic/libc/include/sys/syscall.h
        ~/repo_44/kernel/kernel/ptrace.c
        ~/repo_44/kernel/arch/arm/kernel/ptrace.c
        ~/repo_44/kernel/arch/arm/include/uapi/asm/ptrace.h
        ~/repo_44/android/bionic/libc/arch-arm/include/machine/setjmp.h
        ```
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
    * 参考：futex在android pthread中的应用，[同步应用](./linux.md#sync-application)
    * 参考：pthread相关代码实现
        ```
        ~/repo_ics/android/bionic/libc/arch-arm/bionic/clone.S
        ~/repo_ics/android/bionic/libc/arch-arm/syscalls/_exit_thread.S
        ~/repo_ics/android/bionic/libc/arch-arm/syscalls/__set_tls.S
        ~/repo_ics/android/bionic/libc/bionic/pthread.c
        ~/repo_44/kernel/arch/arm/kernel/ptrace.c
        ~/repo_44/kernel/arch/arm/kernel/calls.S
        ~/repo_44/kernel/kernel/fork.c
        ~/repo_44/kernel/arch/arm/kernel/process.c
        ~/repo_44/kernel/arch/arm/include/asm/thread_info.h
        ~/repo_44/kernel/arch/arm/include/linux/sched.h
        ~/repo_44/kernel/arch/arm/include/asm/processor.h
        ~/repo_44/kernel/arch/arm/include/asm/ptrace.h
        ~/repo_44/kernel/arch/arm/include/asm/tls.h
        ~/repo_44/kernel/include/linux/regset.h
        ```
* exec
    * execve为内核级系统调用，其他（execl，execle，execlp，execv，execvp）都是调用execve的库函数
    * exec系列中，**e代表指定环境变量**，**v指代参数list**,**p指代PATH**
        * The `execv()`, `execvp()`, and `execvpe()` functions provide an array of pointers to **null-terminated strings that represent the argument list** available to the new program.
            * The first argument, by convention, should point to the filename associated with the file being executed. The array of pointers must be terminated by a NULL pointer.
        * The `execle()` and `execvpe()` functions allow the caller to **specify the environment** of the executed program via the argumentenvp.
        * The `execlp()`, `execvp()`, and `execvpe()` functions duplicate the actions of the shell in searching for an executable file if the specified filename does not contain a slash (/) character. The file is sought in the colon-separated list of directory pathnames specified **in the PATH environment variable**.
    * execlp中不能使用重定向(why can't use output redirection in execlp)
        * 比如`execl("/usr/bin/hexdump", "hexdump", "/dev/input/mice", ">", "/dev/ttyS0", (char*) NULL);`
        * 因为execl中没有shell的环境(as execl input has no shell system or environment, if what to redirect, you could)，以下方法可以解决：
            * `system("command > output terminal")`
            * `exel("/system/bin/sh", "sh", "command", ">", "output terminal")` to invoke shell to work
            * use dup to redirect Output redirection using fork() and execl()  execl() + redirecting output to text files
* [chrt](http://man7.org/linux/man-pages/man1/chrt.1.html)
    * chrt change the real-time attributes of a process(priority, schedule type, etc)

<span id="setuid"></span>

* [setuid](http://man7.org/linux/man-pages/man2/setuid.2.html)
    * uid/gid：真实的用户、组（uid、gid）：进程的真正所有者。每当用户在shell终端登录时，都会将登录用户作为登录进程的真正所有者。
    * euid/egid：有效的用户、组（euid、egid）：进程的有效用户、组。进程所执行各种操作所允许的权限（process credentials）是依据进程的有效用户来判断的
    * situation of euid=0(root)
        * `setuid` will permanently changes the uid
        * `seteuid` temporarily changes uid and then use `setuid(0)` to change it back
    * situation of euid>0(not root)
        * `setuid/seteuid` changes its effective uid to only its original euid uid or suid
    * 参考：[sudo&su](../tool/shell.md#sudo-su)，[特殊权限](../tool/shell.md#specialperm)
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
    * Inotify is an internal kernel facility. There is no “inotify file”. There are dedicated system calls `inotify_init`, `inotify_add_watch` and `inotify_rm_watch` that allow processes to register themselves to be notified when certain filesystem events happen.