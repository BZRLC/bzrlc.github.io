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


## 计算TF

```r
trainterm <- trainterm[grepl("\\S", trainterm$term),]
# trainterm[!grepl("\\S", trainterm$term),] ##通过这句可以查看这种空白符
# 例如
# segmentCN("")
trainterm$logic <- rep(1, nrow(trainterm))# 添加辅助列
library(dplyr)
traintfidf <- aggregate(logic ~ id + label + term, data = trainterm, FUN = sum) %>% rename(tf = logic)
total <- length(unique(traintfidf$id))
temp <- data.frame(table(traintfidf$term)/total) 
names(temp) <- c("term", "df")
traintfidf <- left_join(traintfidf, temp)
```

在统计TFIDF等指数之前，还是先要处理下数据，因为在分词的时候分出了空白符，这种空白符即不能用is.na、is.null、is.nan这些函数查出来，也不能使用常见的空白符（空格""，制表符"\t"，换行符"\n"，回车符"\r"，垂直制表符"\v"，分页符"\f"）包括空白符（"\s"）等正则规则查出来，第1行使用非空白符（"\S"）筛除这种变态的情况。除了在分词时会碰到这种情况以外，在抓取网络数据时也会产生。
 
要统计tf，可以通过table函数、dcast函数(reshape2包、plyr包都有这个函数)等实现，但是尝试之后发现它们要不速度慢，要不就是占用内存太高，包括data.table里的dcast函数（data.table包的数据处理速度要比dplyr快10倍以上），原因在于它们的中间过程要进行矩阵的转换，我还是使用添加辅助列的方法来计算tf，不要小看这些雕虫小技，有时候高大上并不等于高效。
这里使用aggregate统计每篇文章每个词的频次，2行添加了一个辅助列logic，当然不添加辅助列，设置aggregate里的FUN参数为length函数也能完成，但是数据量大时耗费时间太长，不如添加辅助列，而FUN参数调用sum函数速度快，这句的意思就是按照id、term、label三列分组后对logic求和。

5行计算统计出参与计算的文章id数，即总文章数；6行的table函数计算文档频数，然后除以总文档数得出文档频率（df）；7行重命名，%>%函数即上文提到过的管道函数，顾名思议作用就是交换两个空间的内容不需要占用其它的空间，减少占用的内存，但是怎么看都很生僻对吧。8行使用dplyr包里的left_join函数将traintfidf与temp左关联，这里需要提醒大家，不要dplyr包、plyr包同时使用，比如这里就会导致rename函数被覆盖，二者的功能相似，没必要同时加载，或者先加载plyr再加载dplyr。

## 计算逆文档频率(IDF)

```r
temp <- data.frame(log(total/(table(traintfidf$term) + 1))) 
names(temp) <- c("term", "idf")
traintfidf <- left_join(traintfidf, temp)
traintfidf$tfidf <- traintfidf$tf*traintfidf$idf
```

1行按照逆文档频率的计算公式计算IDF；2行重命名temp数据框；3行进行左关联，4行计算tfidf，到这里基本上有用的指标都到齐了。

## 随机森林模型

  现在我们有了三个指标：tf、df、tfidf，选哪个用于构建模型？由于tf受高频词影响较大，我们暂时将其排除，根据上面的统计逻辑发现正向样本中某个词语的df和负向样本的相同，因为我们并没有把正负样本分开统计，所以在这种情况下使用df建模基本上不可能将正负样本分开，只有选tfidf了。构建模型时需要将每一个词汇作为一个变量或者维度，这样矩阵会变得异常稀疏，但我们先不讲究这些，在应用方向上做数据挖掘建模时，第一目标不是追求模型统计上的完美性，而是在测试集和训练集上的稳定性和准确性

### 随机森林模型构建及情感分析指数计算

  随机森林的算法我就暂时不提了，原因一是它的资料到处都是，二是因为中间复杂的数学推理过程我也没有弄的很明白。但是这个不影响我使用它，我只用知道它可以用来做回归，分类就行了，R里使用它都是一行代码的事。（其实我把这一部分成两篇来写主要原因就是为了琢磨一下这个随机森林算法的数理知识然后再想想怎么简明扼要的讲出来，不过因为种种原因始终没有时间，不过没有关系，后面我再来。）
  
  这里我是用随机森林做的分类，训练数据标签里只有两个分类1（正向）或-1（负向），所以我的工作理论上属于分类任务，但是情感分析未来肯定要追求执行更加细腻的分类，到达一定程度后最终将由分类任务演化为回归预测任务，另外随机森林算法相对于类似的决策树算法更加高大上，说以我选择了它。（大家现在知道高大上的东西该什么时候用了吧。）
  
  在构建随机森林模型之前，我们首先要把数据调整为randomForest包要求的格式，randomForest函数要求为数据框或者矩阵，需要原来的数据框调整为以每个词作为列名称（变量）的数据框。
  
  
```r
library(reshape2)
train <- dcast(data = traintfidf, id + label ~ term, sum, value.var = "tfidf")
# Error: std::bad_alloc
library(randomForest)
row.names(train) <- train[, "id"]
train <- subset(train, select = -id)
train$label <- as.factor(train$label)
Randommodel100 <- randomForest(x=subset(train, select = -label), y = train[, "label"], importance = TRUE, proximity = FALSE, ntree = 100)#构建模型
print(Randommodel100)
# Call:
# randomForest(x = subset(train, select = -label), y = train[, "label"], ntree = 100, importance = TRUE, proximity = FALSE) 
# Type of random forest: classification
# Number of trees: 100
# No. of variables tried at each split: 157
# 
# OOB estimate of error rate: 7.04%
# Confusion matrix:
# -1 1 class.error
# -1 11602 274 0.02307174
# 1 968 4808 0.16759003
```

2行将原来的long型数据框转化为wide型数据框（前面已经讲过），data.table里的dcast函数比reshape2包里的dcast好用，尽管他们的参数都一样，但是很多人还是比较喜欢老朋友reshape2包(比如我),如果你的电脑报告内存不足的错误，可以使用data.table包里的dcast函数试试；4、5行用于移除id列，因为id列不是随机森林模型所需要的数据，先将id列赋值给行编号，然后移除，这样既对原文本做了标记，同时移除了建模无用的数据；6行将标签列转化为因子，randomForest函数在执行建模任务时，首先判断因变量的类型，如果因变量是因子则执行分类任务，如果因变量是连续性变量，则执行回归预测任务；7行执行建模，x参数设定自变量数据集，y参数设定因变量数据列，importance设定是否输出因变量在模型中的重要性，如果移除某个变量，模型方差增加的比例是它判断变量重要性的标准之一，proximity参数用于设定是否计算模型的临近矩阵，ntree用于设定随机森林的树数（后面再讨论），最后一句输出模型在训练集上的效果。

从上面的结果中看到，整体准确率达到90%以上，随机森林对训练集的分类还是不错了，另外，那些说随机森林不可能过度拟合的人都是骗子，为了各自的需求人们习惯断章取义，在任何领域都同样存在这种一知半解或不无恶意的人，我们不妨验证一下上面的随机森林模型是否发生了过拟合，所谓过拟合就是模型在训练集上比较牛逼，在测试集上就蔫了的现象。
 
好了，文本分析的内容在我这里就结束了，因为文本分析并不是我的主题，它只是某种特殊格式数据的数据分析而已。当然了，情感分析只是文本分析的皮毛，还有丧心病况的语义分析，高大上的语言自动应答等大山一样的领域横在文本分析的前面。

张志刚老师就和我说过，文本分析的一个毛病就是没有明确的问题。的确，情感分析的结果或者说我算出来的结果有什么意义呢，这也是个问题，我们只能从结果看看有多少评论骂了你，多少夸了你。我不能说这个结果有什么意义，但是这些方法和技巧的确是非常流行，我只能认为对于很多情况来说，这样就够了。

