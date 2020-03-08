
[toc]

[programming](./prog.md)

<span id="huffman"></span>
* [huffman coding(霍夫曼编码)](https://zh.wikipedia.org/wiki/%E9%9C%8D%E5%A4%AB%E6%9B%BC%E7%BC%96%E7%A0%81)
    * 介绍：最优二叉树，是一种带权路径长度最短的二叉树。
    * 带权路径最短：树的路径长度是从树根到每一结点的路径长度之和，记为WPL=（W1*L1+W2*L2+W3*L3+...+Wn*Ln），N个权值Wi（i=1,2,...n）构成一棵有N个叶结点的二叉树，相应的叶结点的路径长度为Li（i=1,2,...n）。可以证明霍夫曼树的WPL是最小的。

    ![Huffman_algorithm](./algorithm/Huffman_algorithm.gif)
    ![huffman](./algorithm/huffman.jpg)