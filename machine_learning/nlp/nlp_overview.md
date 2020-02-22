
[toc]

[main page](../entry.md)

### 常见的NLP任务及对应数据集
|name|full name|type|subtype|comment|
|---|---|---|---|---|
|SQuAD|Standford Question Answering Dataset|token-level task||样本为语句对. 给出一个问题, 和一段来自于Wikipedia的文本, 其中这段文本之中, 包含这个问题的答案, 返回一短语句作为答案.|
|NER|Named Entity Recognition|token-level task||对句子中的每个token打标签, 判断每个token的类别.|
|MNLI|Multi-Genre Natural Language Inference|sequence-level task|Natural Language Inference|GLUE Datasets 推断两个句子是意思相近, 矛盾, 还是无关的.|
|QQP|Quora Question Pairs|sequence-level task|Sentence Pair Classification|判断两个来自于Quora的问题句子在语义上是否是等价的|
|QNLI|Question Natural Language Inference|sequence-level task|Sentence Pair Classification|二分类问题, 两个句子是一个(question, answer)对. 正样本为answer是对应question的答案, 负样本则相反.|
|STS-B|Semantic Textual Similarity Benchmark|sequence-level task|Sentence Pair Classification|类似回归的问题. 给出一对句子, 使用1~5的评分评价两者在语义上的相似程度.|
|MRPC|Microsoft Research Paraphrase Corpus|sequence-level task|Sentence Pair Classification|句子对来源于对同一条新闻的评论. 判断这一对句子在语义上是否相同.|
|RTE|Recognizing Textual Entailment|sequence-level task|Sentence Pair Classification|二分类问题, 类似于MNLI, 但是数据量少很多|
|SST-2|Stanford Sentiment Treebank|sequence-level task|Single Sentence Classification|单句的二分类问题, 句子的来源于人们对一部电影的评价, 判断这个句子的情感.|
|CoLA|Corpus of Linguistic Acceptability|sequence-level task|Single Sentence Classification|单句的二分类问题, 判断一个英文句子在语法上是不是可接受的.|
|SWAG|Situations With Adversarial Generations|sequence-level task||给出一个陈述句子和4个备选句子, 判断前者与后者中的哪一个最有逻辑的连续性, 相当于阅读理解问题.|

* [自然语言处理全家福：纵览当前NLP中的任务、数据、模型与论文](https://zhuanlan.zhihu.com/p/38445982)  
* [Tracking Progress in Natural Language Processing](https://github.com/sebastianruder/NLP-progress)  

### 常见的NLP数据增强方法
1. 同义词替换（SR: Synonyms Replace）：不考虑stopwords，在句子中随机抽取n个词，然后从同义词词典中随机抽取同义词，并进行替换。  
2. 随机插入(RI: Randomly Insert)：不考虑stopwords，随机抽取一个词，然后在该词的同义词集合中随机选择一个，插入原句子中的随机位置。该过程可以重复n次。  
3. 随机交换(RS: Randomly Swap)：句子中，随机选择两个词，位置交换。该过程可以重复n次。  
4. 随机删除(RD: Randomly Delete)：句子中的每个词，以概率p随机删除。

参考  
* [NLP中一些简单的数据增强技术](https://zhuanlan.zhihu.com/p/63182132)  

### 常见的NLP数据清洗方法
1. 缩略词更改  
2. 拼写校正  
3. 标点符号  
4. 符号替换  
5. 去除空格  
