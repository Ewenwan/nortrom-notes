
[toc]

[main page](../entry.md)

# 语法

* [Shell编程基础](http://wiki.ubuntu.org.cn/Shell%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80)

# linux使用基本概念

## 文件系统

* RWX
    * 目录
        * r权限：拥有此权限表示可以读取目录结构列表，也就是说可以查看目录下的文件名和子目录名，注意：仅仅指的是名字。
        * w权限：拥有此权限表示具有更改该目录结构列表的权限。
        * x权限：拥有目录的x权限表示用户可以进入该目录成为工作目录。
    * 文件
        * r权限：用于此权限表示可以读取此文件的实际内容。
        * w权限：拥有此权限表示可以编辑、添加或者是修改该文件的内容。但是不包含删除该文件（由目录权限决定）。
        * x权限：表示该文件具有可以被系统执行的权限。
* atime/ctime/mtime
    * 文件
        * atime(Access time)是在读取文件或者执行文件时更改的任何对inode的访问都会使此处改变。
        * mtime(Modified time)是在写入文件时随文件内容的更改而更改的。
        * ctime(Change time)是在写入文件、更改所有者、权限或链接设置时随 Inode 的内容更改而更改的。只要stat出来的内容发生改变就会发生改变。mtime的改变必然导致ctime的改变。
    * 目录
        * atime(Access time)是在读取文件或者执行文件时更改的（我们只cd进入一个目录然后cd ..不会引起atime的改变，但ls一下就不同了）。
        * mtime(Modified time)是在目录中有文件的新建、删除才会改变（如果只是改变文件内容不会引起mtime的改变。
        * ctime(Change time)基本同文件的ctime，其体现的是inode的change time。

## 设备

* /dev/tty
    * /dev/tty1 设备文件，通常使用tty来简称各种类型的终端设备。tty是Teletype的缩写。
    * /dev/ttyn, /dev/console 连接显示器登陆控制台
        * 在Linux系统中，计算机显示器通常被称为控制台终端或控制台（Console）。它仿真了类型为Linux的一种终端，并且有一些设备特殊文件与之相关联：tty0、tty1、tty2等。当你在控制台上登录时，使用的是tty1。使用Alt+[F1—F6]组合键时，我们就可以切换到tty2、tty3等上面去。tty1 –tty6等称为虚拟终端，而tty0则是当前所使用虚拟终端的一个别名。Linux系统所产生的信息都会发送到该终端上。因此不管当前我们正在使用哪个虚拟终端，系统信息都会发送到我们的屏幕上
    * /dev/ttySn 用串口登陆串行端口终端
        * 串行端口终端（Serial Port Terminal）是使用计算机串行端口连接的终端设备。计算机把每个串行端口都看作是一个字符设备。/dev/ttyS0和/dev/ttyS1分别对应COM1和COM2。可以给linux安装minicom调试设备。
    * /dev/pts/n 伪终端
        * **用putty或者SecureCRT登陆伪终端**，用w或者who命令。**常用的ctrl+alt+T命令开启的terminal**。伪终端或者虚拟终端（Pseudo Terminal）是成对的逻辑终端设备，pts就是定义虚拟终端的

## 网络

* sshfs与nfs
    * 相同点：都能mount到本地目录并对文件进行操作
    * sshfs特点：sshfs安全性更好，效率偏低，适合局域网和WAN。sshfs挂载后，客户user对挂载目录的操作会被ssh对应的服务器用户授权，所有更改或创建均被认为是服务器用户所为。对权限要求高的场合十分重要。mount nfs则不然
    * nfs特点：nfs效率高适合局域网，没有安全性（除非经过很好配置）

# 命令

## 包管理

* list all the package that installed
    * `sudo dpkg -l | more`
* [list the data and version installed](http://askubuntu.com/questions/389715/how-to-list-installed-package-and-its-details-on-ubuntu)
    * `cat /var/log/dpkg.log | grep "install"`
* list where the package installed
    * `dpkg -L packagename`
* list the runnable binary's location and which version it used location "which binary"
    * `dpkg --listfiles packagename`
* how many that kind of binary
    * `whereis binary`
* how to uninstall the installed package
    * `dpkg -r packagename`


## 环境和常用

* \$\# \$\@ \$\*
    * 脚本名称叫test.sh 入参三个: 1 2 3，运行test.sh 1 2 3
        * \$\*为"1 2 3"（一起被引号包住）
        * \$\@为"1" "2" "3"（分别被包住）
        * \$\#为3（参数数量）
* PATH
    * ``export PATH="$PATH:/home/someone/bin"``
* source
    * source FileName。作用:在当前bash环境下读取并执行FileName中的命令。
    * 该命令通常用命令“.”来替代。如：`source /etc/profile` 与 `./etc/profile`是等效的。
    * source命令与shell scripts的区别是，source在当前bash环境下执行命令，而scripts是启动一个子shell来执行命令。这样如果把设置环境变量（或alias等等）的命令写进scripts中，就只会影响子shell,无法改变当前的BASH,所以通过文件（命令列）设置环境变量时，要用source命令。
* 压缩解压
    * ``tar -zxvf ....tar.gz ``
    * ``tar -jxvf ...tar.bz2``
    * ``zip -r yasuo.zip abc.txt dir1``：把一个文件abc.txt和一个目录dir1压缩成为yasuo.zip
    * ``unzip yasuo.zip``
* 硬链接和符号链接
    * 简单的说:硬连接记录的是目标的inode,符号连接记录的是目标的path。
    * 软连接就像是快捷方式,而硬连接就像是备份!符号连接可以做跨分区的 link；而 硬连接由于 inode 的缘故，只能在本分区中做 link.所以,符号连接的使用频率要高的多。
* bash(/bin/bash) & dash(/bin/sh)
    * dash 比 bash 更轻，更快。但 bash 却更常用。
    * Ubuntu的 shell 默认安装的是 dash，而不是 bash。
    * 运行以下命令查看 sh 的详细信息，确认 shell 对应的程序是哪个：
    
    ```
        $ls -al /bin/sh
    ```

    * Linux 中的 shell 有很多类型，其中最常用的几种是: Bourne shell (sh)、C shell (csh) 和 Korn shell (ksh), 各有优缺点。**Bourne shell 是 UNIX 最初使用的 shell**，并且在每种 UNIX 上都可以使用, 在 shell 编程方面相当优秀，但在处理与用户的交互方面做得不如其他几种shell。Linux 操作系统缺省的 shell 是Bourne Again shell，它是 Bourne shell 的扩展，简称 **Bash，与 Bourne shell 完全向后兼容，并且在Bourne shell 的基础上增加、增强了很多特性**。Bash放在/bin/bash中，它有许多特色，可以提供如命令补全、命令编辑和命令历史表等功能，它还包含了很多 C shell 和 Korn shell 中的优点，有灵活和强大的编程接口，同时又有很友好的用户界面。GNU/Linux 操作系统中的 /bin/sh 本是 bash (Bourne-Again Shell) 的符号链接，但鉴于 bash 过于复杂，**有人把 ash 从 NetBSD 移植到 Linux 并更名为 dash (Debian Almquist Shell)，并建议将 /bin/sh 指向它**，以获得更快的脚本执行速度。Dash Shell 比 Bash Shell 小的多，符合POSIX标准。

## 网络

* ``who -r``
    * 查看当前的Linux服务器的运行级别
* ``route -n`` & ``netstat -nr``
    * 查看默认网关。除了默认的网关信息，这两个命令还可以显示当前的路由表。
* `scp`
    * scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令，适用于本地到服务器的拷贝，或者服务器之间的拷贝，且可以很好的解决权限问题。
    * e.g.: `scp -r ~/.ssh xj05@10.125.12.180:~/  `

## 系统

## 文本

参考:[搞定Linux Shell文本处理工具，看完这篇集锦就够了](https://zhuanlan.zhihu.com/p/29718871)

* 单引号双引号反引号
    * 单引号和双引号，都是为了解决中间有空格的问题。他们的区别在于，单引号将剥夺其中的所有字符的特殊含义，而双引号中的'$'（参数替换）和'`'（命令替换）是例外。如果需要在双引号””里面使用这两种符号，需要用反斜杠转义。
    * 反引号``和\$()等价
* \>和\>\>
    * \> 是定向输出到文件，如果文件不存在，就创建文件；如果文件存在，就将其清空；一般我们备份清理日志文件的时候，就是这种方法：先备份日志，再用`>`，将日志文件清空（文件大小变成0字节）；
    * \>>这个是将输出内容追加到目标文件中。如果文件不存在，就创建文件；如果文件存在，则将新的内容追加到那个文件的末尾，该文件中的原有内容不受影响。
* `patch` & `diff`
    * patch命令就是用来将修改（或补丁）写进文本文件里。patch命令通常是接收diff的输出并把文件的旧版本转换为新版本。
        ```
        diff -Naur old_file new_file > diff_patch
        patch -p0 old_file diff_patch
        ```
* grep
    * ``grep -rn "text" filename``：递归搜索
    * ``grep -c "text" filename``：统计文件中包含文本的次数
    * ``grep -i "text" filename``：忽略大小写
    * ``grep -e "class" -e "vitural" file``：匹配多个模式
* sort
    * -n 按数字进行排序 VS -d 按字典序进行排序
    * -r 逆序排序
    * -k N 指定按第N列排序
    * ``sort -nrk 1 data.txt``：第二列按数字逆序排序
    * ``sort -bd data``：忽略像空格之类的前导空白字符
* uniq
    * 消除重复行
    * ``sort unsort.txt | uniq``：消除重复行
    * ``sort unsort.txt | uniq -c``：统计各行在文件中出现的次数
    * ``sort unsort.txt | uniq -d``：找出重复行
* tr
    * ``cat text| tr '\t' ' '``：制表符转空格
    * ``cat file | tr -d '0-9'``：删除所有数字
    * ``cat file | tr -c '0-9'``：获取文件中所有数字
    * ``cat file | tr -s ' '``：压缩字符（压缩多余的空格）
    * ``cat file | tr '[:lower:]' '[:upper:]'``：小写转大写
* sed
    * ``seg 's/text/replace_text/' file``：替换每一行的第一处匹配的text
    * ``seg 's/text/replace_text/g' file``：全局替换
    * ``sed '/^$/d' file``：移除空白行
    * ``echo this is en example | seg 's/\w+/[&]/g'$>[this] [is] [en] [example]``：变量转换，已匹配的字符串通过标记&来引用.
* wc
    * ``wc -l file``：统计行数
    * ``wc -w file``：统计单词数
    * ``wc -c file``：统计字符数
* awk
    * 脚本结构：``awk ' BEGIN{ statements } statements2 END{ statements } '``
    * 工作方式：1.执行begin中语句块；2.从文件或stdin中读入一行，然后执行statements2，重复这个过程，直到文件全部被读取完毕；3.执行end语句块；
    * 特殊符号
        * NR:表示记录数量，在执行过程中对应当前行号；
        * NF:表示字段数量，在执行过程总对应当前行的字段数；
        * \$0:这个变量包含执行过程中当前行的文本内容；
        * \$1:第一个字段的文本内容；
        * \$2:第二个字段的文本内容；
    * ``awk '{print $2, $3}' file``：打印每一行的第二和第三个字段
    * ``awk ' END {print NR}' file``：统计文件的行数
    * ``echo -e "1\n 2\n 3\n 4\n" | awk 'BEGIN{num = 0 ;
print "begin";} {sum += $1;} END {print "=="; print sum }'``：累加每一行的第一个字段
    * ``awk -F ' ' '{print $NF}' /etc/passwd``：设置定界符为空格


## 搜索

* ``find``
    * ``find /usr -size +10M``：在/usr目录下找出大小超过10MB的文件
    * ``find /home -mtime +120``：在/home目录下找出120天之前被修改过的文件
    * ``find /var ! -atime -90``：在/var目录下找出90天之内未被访问过的文件
    * ``find / -name core -exec rm {} ;``：在整个目录树下查找文件“core”，如发现则无需提示直接删除它们
    * ``find . \( -name "*.txt" -o -name "*.pdf" \)``：查找txt和pdf文件
    * ``find . -type f -perm 644``：找具有可执行权限的所有文件
    * ``find . -type f -perm 644 -print``：找具有可执行权限的所有文件（-print0 使用'\0'作为文件的定界符，这样就可以搜索包含空格的文件）