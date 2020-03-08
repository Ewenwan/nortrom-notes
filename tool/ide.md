
[toc]

[tool](./tool.md)

# terminal

* linux 修改终端界面大小
    * 在~/.bashrc文件末尾加入如下命令:其中rows为终端显示的行数cols为列数
    ```
    resize -s rows cols
    ```
* [linux 修改终端字体颜色](http://blog.chinaunix.net/uid-26021340-id-3481924.html)
    * vim ~/.bashrc。修改和PS1有关的部分
    ```
    PS1='\[\033[4;31;40m\]\u\[\033[00m\]@\h:\[\033[37;40m\]\w\[\033[32;40m\]\$ \[\033[34;40m\]'
    ```


# vscode

* 同时编辑多行
    * Alt+Shift 竖列选择