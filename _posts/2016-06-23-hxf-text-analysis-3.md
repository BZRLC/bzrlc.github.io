---
title: "文本分析初探-3"
author: hxf
date: "23 June 2016"
layout: single
comments: true
categories: [NLP]
tags: [NLP,R]
mathjax: true
highlight: true
description: "文本分析处理 & TF-IDF简介"
---

 现在我来介绍另一种方法来作为文本分析这个部分的结尾。文本分析的算法除了词典法就是比较流行的监督式模型了，这个领域你可以使用各种高大上的算法来提升逼格，什么SVN，决策树，随机森林，神经网络等；但效果怎么样还真的有待检验。 

一个完整的监督式分析模型一般分为以下步骤：

1. 训练库构建，监督式过程必须通过训练集帮助模型识别特征；
2. 特征提取及模型构建，比如TFIDF算法；
3. k层交叉检验进行结果评价。

## TF-IDF指标

既然决定使用监督模型，就要有一套衡量特征变化的指标。比如可以使用TF词频作为变量也可以使用DF文档频率来作为变量，总之在模型构建之前需要构建几个指标。我之前说过，计算机是无法识别词义的，它能做的事情只有计算，而我们要使用计算机来做文本分析的话，就得设计算法，通过计算结果来反映词义，在这里，指标的选取不可避免，我们不妨暂定TF、DF和TFIDF作为指标，为什么要选取它们作为指标呢，这和使用者的水平关系很大。

现在来帮助大家理解一下TF，IDF这两个指标，比如有一篇《中国环境资源报告的文章》，提取这篇文章的关键词比较简单有效的方法就是根据词频（Term Frequency，缩写为TF）来提取。TF就是一个文章中出现某个词的次数（当然如果进行文章之间的横向比较，因为文章有长有短，还要除以文章的总词数）。你可能认为这篇文章中“中国”出现的频率会最高，其实不是，“的”，“是”，“在”，“地”，这些词出现的频率才是最高的，这类词是停用词，在提取关键词之前必须剔除掉。

$$ TF= \frac{某词在文章中出现的次数}{文章包含的总词数} $$

   剔除之后‘中国’就成了出现最多的词，但是‘中国’在所有的文章中出现的可能性都很大，而‘环境’、‘资源’、才是更关键的关键词，所以要给TF一个权重。于是"逆文档频率"（Inverse,Document,Frequency，缩写为IDF）横空出世，但首先需要知道文档频率（Document,Frequency）是个什么玩意，二者计算计算公式如下：

$$ DF = \frac{包含某词的文档数}{语料库的文档总数} , IDF = \log(\frac{语料库的文档总数}{包含某词的文档数 + 1}) $$

  如果一个词比较“常见”（指在日常所有文档中），那么它的IDF就比较低。要计算IDF，首先要有一个充实的语料库。利用IDF作为惩罚权重，就可以计算词的TF-IDF。

$$ TF-IDF = TF \times IDF $$

  这样较常见的“中国”在《中国环境资源报告》这篇文章中的TFIDF就会被拉底，而其他稀有词汇的TFIDF就会相对提高，这样我们在按照TFIDF排序即可以方便的找出关键词了。
  通过计算TF、DF、TFIDF三个指标构建语料库，至少保证了监督模型输入变量，可以选择其中一种或多种作为指标构建模型。

## 构建语料库  
  在建模之前，应该使用训练集构建一个语料库，语料库至少包括以下几个维度：文本id、词汇、TF、DF和TFIDF。首先要将训练集清洗一下，清洗过程基本和前面的词典法相同。
  以下都是构建语料库的过程。


### 训练集分词预处理

```r
train<-read.csv("F:/study/新建文件夹/rawdata/review_sentiment/train.csv",stringsAsFactors=F)
sentence <- as.vector(train$word)
sentence <- gsub("[[:digit:]]*", "", sentence) 
sentence <- gsub("[a-zA-Z]", "", sentence)
sentence <- gsub("\\.\\.", "", sentence)
train <- train[!is.na(sentence), ]
sentence <- sentence[!is.na(sentence)]
train <- train[!nchar(sentence) < 2, ]
sentence <- sentence[!nchar(sentence) < 2]
```
  这里的train数据集就是我们第一篇创建的数据集，上次我保存了，这次只用再读取一遍就行了，不知道它是怎么来的自行看第一篇。用read.csv读取文件时，有时可能会报警‘EOF within quoted string”，一般是因为数据中不正常符号导致，一般为数据中不正常的符号所致，常见的方法是将quote = ""设置为空，这样做虽然避免了警告，但是仍然解决不了问题，有时数据会对不上号，所以最好从符号上着手将一些特殊符号去除，这些语句我之前也说过了，这里不再提了。
  
### 训练集分词

```r
library(Rwordseg)
dict<-read.csv("F:/study/新建文件夹/dict/dict.csv")
dict<-as.character(dict)
insertWords(dict)
system.time(x <- segmentCN(strwords = sentence)) 
temp <- lapply(x, length)
temp <- unlist(temp)
id <- rep(train[, "id"], temp)
label <- rep(train[, "label"], temp)
term <- unlist(x)
trainterm <- as.data.frame(cbind(id, term, label), stringsAsFactors = F)
```

分词过程和上节讲的一样，另外还沿用了上节的情感词典dict，不清楚怎么来的同样自行查看上一节。

### 去除停用词  

```r
stopword<-read.csv("F:/study/新建文件夹/dict/stopword.csv",header=T,sep=",",stringsAsFactors=F)
stopword <- stopword[!stopword$term %in% dict,]
trainterm <- trainterm[!trainterm$term %in% stopword,]
```

  在使用复杂的算法时，尽量去除一些非特征词汇可以有效的降低计算量和内存占用率，尽管词典法里面去除停用词这个环节可有可无，但是这里将要用到其他复杂的算法，去除停用词是必须环节，因此使用上节整理的情感词典去除停用词中具有情感色彩的成分，然后对分词结果去除停用词。


