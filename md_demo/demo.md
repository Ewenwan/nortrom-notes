
[toc]

[main page](../entry.md)

# Markdown使用笔记
## font-part
正常字体
*斜体*  
**粗体**  
***斜粗体***  
~~横线~~  
<u>下划线</u>  
footnote[^note1]  
[^note1]:information of first note  

separator:  
****
* [Markdown 段落](https://www.runoob.com/markdown/md-paragraph.html)
  
## seg-part
list:
1. first
    * something-a
    * something-b
2. second
3. third
nested segment:  
> a
> > b
> > c
link: <cn.bing.com>  
reference or knowledge points of this part:  
* [Markdown 列表](https://www.runoob.com/markdown/md-lists.html)
* [Markdown 区块](https://www.runoob.com/markdown/md-block.html)
* [Markdown 链接](https://www.runoob.com/markdown/md-link.html)
  
## code-part
in line code `printf()`  
out of line code:  
```python
for item in a:
    b[i] = item
```
reference or knowledge points of this part:  
* [Markdown 代码](https://www.runoob.com/markdown/md-code.html)
  
## table-part
|a|b|c|
|:---|:---:|---:|
|thing1|thing2|thing3|
  
## img-part
using markdown keyword:  
![img1](./data/languages_cpp.png)  
using html keyword:  
<img src="./data/languages_cpp.png" width=50%>  
reference or knowledge points of this part:
* [Markdown 图片](https://www.runoob.com/markdown/md-image.html)  
  
## equation-part
in line equation:$f(x)$
out line equation:
$$
a = b + \frac1c + \bar{a} + \hat{a} + \epsilon
$$
$$
a_{ij} = \sum_{k}^N b_{ijk} \tag{1}
$$
$$
\int_0^\infty{f(t)dt} \tag{2}
$$
$$
a = 
\begin{bmatrix}
a & b & c \\
1 & 2 & 3 \\
\end{bmatrix}
\tag{3}
$$
reference or knowledge points of this part: 
* katex is a fast implementation for latex and usually used in web application
* katex only support a subset of latex grammar(e.g. labeling not supported)
* [LaTeX 第五课：数学公式排版](https://zhuanlan.zhihu.com/p/24502400)
* [Latex的公式输入](https://www.jianshu.com/p/05987743d27c)
  
## graph-part
this is a flow graph of basic AI application working flow
```flow
st=>start: img
op1=>operation: detect
cond=>condition: has object?
op2=>operation: preprocess
op3=>operation: recognition
end=>end: finish
st->op1->cond
op2->op3->end
cond(yes)->op2
cond(no)->end
```
reference or knowledge points of this part:  
* [VSCode+Markdown+UML](https://zhuanlan.zhihu.com/p/53087164)
* [Markdown 高级技巧](https://www.runoob.com/markdown/md-advance.html)
* Markdown Preview Enhanced can't show mermaid normally
  
## summary
* 除了vscode+markdown+git的管理模式外，还有几种笔记管理方法，分别是基于markdown的[typora](https://typora.io/)(图床不好用)，号称all-in-one的notion(付费)，以及经典的evernote
* vscode需要安装的markdown插件包括:  
  * Markdown All in One
  * Markdown Preview Enhance
  * GitLens
  * (非必须)Markdown Shortcuts
  * (非必须)Markdown Preview Mermaid Support
  * (非必须)PicGo
* (非必须)chrome需要安装的markdown插件包括：  
  * Markdown Viewer
* git & vscode