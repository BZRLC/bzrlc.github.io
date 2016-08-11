---
title: "文本分析初探-2"
author: hxf
layout: single
author_profile: true
read_time: true
comments: true
---


## 情感分析简介

文章接着上文，代码也接着上文。我当然没有理由把数据整理一下就不管了，对于暂时无法自己爬数据的我来说这些数据可是非常珍贵的。有人可能发现，文本数据的整理方法和字符处理的方法非常相似。没有错，文字对于人来说是博大精深的文化象征，我们看到文字就会自然而然的联想起相印的信息，但是计算机不会，它眼里的文字和我们经常看到的未知数X,Y差不多，就是用来表达意义的一些符号，当然这些意义是计算机识别不了的，我们需要人为的编写算法来得到文本的意义。

这里我要使用的方法是词典法，很好理解。我们需要找到专业人士编写的情感词典，将某种语言中用于表达情感的词汇分成两个类别，正向和负向，然后用对比文本中正向和负向词汇数量之类的方法来判断文本的情感倾向。首先我们应感感谢一些前辈的积累工作，就词典方面中国知网的研究者整理出知网情感hownet词典，更新地址:<http://www.keenage.com/html/c_bulletin_2007.htm>. 另外还有台湾大学整理的台湾大学情感NTUSD，还有富士通公司的情感词典、程度词典等等，这里我们使用的词典就是知网的词典。

## 工欲善其事，必先利其器

进行文本分析首先要准备好工具，R和python都是文本挖掘的利器，虽然我使用的是R，但是如果去搜索自然语言的处理的话，的确是关于python的比较多。并不是说有了R就可以做文本分析了，你还需要安装一个文本分词包比如Rwordseg那么现在第一个难题就来了，Rwordseg包的安装。
 我是弄不明白这些幺蛾子是干什么的，Rwordseg包的安装也确实是折磨了我很长世间，我的理解是很多编程的环境里对中文非常不友好，所以我们要调用一个强大的开发工具Java来解决环境的问题。

### rJava包的配置

首先RJava包需要安装，它是R语言和Java的通信接口，允许在R中直接调用Java的对象和方法，能满足很多包的需要，比如Rwordseg。然后去oracle官网下载JDK版安装包，注意，这里是jdk不是jre，是64位还是32位要和R版本相同，Java也是开源，自己去找，很好下。

完成以上工作以后需要配置环境，右键我的电脑——高级系统设置——环境变量，在环境变量中分别新建或添加相应的环境路径。

第一个classpath，新建classpath添加下面代码


```r
%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;C:\Program;

Files\R\R-3.2.1\library\rJava\jre
```

第二个新建`JAVA_HOME`, 然后把下面的内容添加进去 `C:\Program Files\Java\jdk1.8.0_51`

第三个Path，新建path,注意一下所有的软件都要区分i386或x64


```r
%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;C:\ProgramFiles\R\R-3.2.1\bin\i386;C:\ProgramFiles\R\R-3.2.1\bin\x64;C:\Program Files\Java\jdk1.8.0_51\jre\bin\server
```

第四个R_HOME，添加 `C:\Program Files\R\R-3.2.1` .

以上除了路径均改为是你自己的软件安装路径，其他不需要修改。这些代码可不是R的代码，是我的电脑里的环境环境变量。


```r
if (!suppressWarnings(require("rJava"))) {
install.packages("rJava")
require("rJava")
}
.jinit()
s <- .jnew("java/lang/String", "Hello World!")
s
[1] "Java-Object{Hello World!}
```
如果s正常返回`"Java-Object{HelloWorld!}"`则RJava已经成功了。只有RJava配置成功了，Rwordseg安装才可能成功，前者是后者的依赖包。如果返回不正常说明你的RJava还没安装好，需要重新安装

### 安装Rwordseg


```r
install.packages("Rwordseg")
library(rJava)
library(Rwordseg)
segmentCN("来看看分词包能不能用 ")
[1] "来"   "看看" "分词" "包"   "能"   "不能" "用"
```

如果返回这个结果，说明Rwordseg安装成功了，否则就要自求多福了。

没错，除了这个部分的代码是我自己在R里运行然后复制过来的之外，其它的都是我又百度了一编然后整理出来的，因为我也不知道这都是啥啥啥，甚至我都不知道自己是怎么安装好的。

当然了R里还有其它的分词包，比如jiebaR,这个包不错，小巧玲珑，也没有那么多幺蛾子，而且R和python的社区里都有，关于jiebaR的详细资料：<https://qinwenfeng.com/jiebaR/>

上次好像说了数据的前期处理结束了，其实不是，是我弄错了，只是处理方法说完了而已，而且我好像用错数据了，那个train2里的文本是为算法分析法准备的，算然词典法也是一种算法。不过不要紧，可以再来一遍。
  



```r

#获取文本路径
reviewpath<-'F:/study/新建文件夹/rawdata/review_sentiment/test2'
completepath <- list.files(reviewpath, pattern = "*.txt$", full.names = TRUE)

#批量读入文本
read.txt <- function(x) {
    des <- readLines(x)
    return(paste(des, collapse = ""))
}
review <- lapply(completepath, read.txt)

#list转数据框
docname<-list.files(reviewpath,pattern="*.txt$")
reviewdf <- as.data.frame(cbind(docname, unlist(review))  , stringsAsFactors = F) 
colnames(reviewdf) <- c("id", "words")

#正则替换掉空格
reviewdf$words <- gsub(pattern = "\\s", replacement ="", reviewdf$words)

#正则去掉标点符号
reviewdf$words <- gsub(pattern = "[[:punct:]]", replacement = "", reviewdf$words)
```

### 关联标注


```r
reviewclass<-read.table("F:/study/新建文件夹/rawdata/review_sentiment/test2.rlabelclass", stringsAsFactor = F)
colnames(reviewclass) <- c("id", "label")

if (!suppressWarnings(require("plyr"))) {
    install.packages("plyr")
    require("plyr")
 }

reviewdf <- join(reviewdf, reviewclass)
reviewdf <- reviewdf[!is.na(reviewdf$label),]
test <- reviewdf
```

第1行read.table读取数据，read.table是R读取数据的根函数，碰到了不认识的格式，可以尝试用它读取。第2行文件命名第3行用if构造了一个函数，如果你的电脑里已经有plyr包则执行加载命令，否则执行先下载后加载命令，也就是说不管三七二十一只要把它运行了就可以了。第4行合并文件集，第5行除去缺失值，虽然plyr包已经断更，取而待之的是dplyr包，但我还是对plyr更情有独钟，因我我最先接触的是plyr，毕竟糟糠之妻不可弃。其实dplyr包只是换了种编码格式提高了运算速度而已，数据量不大它们并没有什么嘛区别。在运算速度方面其实dplyr包也有点落伍了。第6行我们的测试数据集就准备好了。

### 词典整理


```r
dictpath_pos<-"F:/study/新建文件夹/rawdict/posdic"
dictpath_neg<-"F:/study/新建文件夹/rawdict/negdic"
completepath_pos <- list.files(dictpath_pos, pattern = "*.txt$", full.names = TRUE)
completepath_neg <- list.files(dictpath_neg, pattern = "*.txt$", full.names = TRUE)
dict_pos <- lapply(completepath_pos, readLines)
dict_neg <- lapply(completepath_neg, readLines)
dict_pos <- unique(unlist(dict_pos))
dict_neg <- unique(unlist(dict_neg))
pos <- dict_pos
neg <- dict_neg
```

这串代码我们分别读取了正负向情感词词典，再用unlist将列表解散为向量，词典比较杂，里面肯定有重复的词，所以还要分别去重，这样词典的正负向词汇就整理也完成了。

## 分词整理

基于词典的情感分析实际上算法和模型先入为主的预订了：统计文本正负情感词的得分之和，如果得分为正，则文本情感倾向为正，反之则为负。所以不需要训练模型，直接使用测试集测试一下即可。基于以上，首先要进行中文分词，在分词之前要将文本预处理一下，包括清除一些英文和数字等等。

### 分词预处理


```r
sentence <- as.vector(test$words)
sentence <- gsub("[[:digit:]]*", "", sentence)
sentence <- gsub("[a-zA-Z]", "", sentence)
sentence <- gsub("\\.", "", sentence)
test <- test[!is.na(sentence), ]
sentence <- sentence[!is.na(sentence)]
test <- test[!nchar(sentence) < 2, ]
sentence <- sentence[!nchar(sentence) < 2]
```

第1行将内容转化为向量sentence，第2行清除数字，第3行清除字母，有些文本可能是一些外国人或者装逼汉写的评价，全是英文，我们无法分析，也要删除。经过这几步处理有些文本只剩下dot符号了，第4行将它们清除。有些文本处理后可能就成了空值或者少于2个字符（2个字符以下的文本没有分析的价值），所以5，6，7行将这些文本分别删除。

另外既然整合了大量的词典，就要尽量保证分词器能够把这些情感词汇分出来，所以需要将情感词典添加到分词器的词典中去，虽然这种方法在特殊情况下并不一定凑效，但至少增加了分词器在分词时的计算权重。

### 添加词典


```r
pos<-as.data.frame(pos)
neg<-as.data.frame(neg)
weight <- rep(1, length(pos[,1]))
pos <- cbind(pos, weight)
weight <- rep(-1, length(neg[,1]))
neg <- cbind(neg, weight)
posneg <- rbind(pos, neg)
names(posneg) <- c("term", "weight")
posneg <- posneg[!duplicated(posneg$term), ]
dict <- posneg[, "term"]
library(Rwordseg)
insertWords(dict)
```
1，2行将正负情感词转化为数据框（便于后来操作）。3，4，5，6行给正负情感词添加权重。这里正负向情感词权重分别为为+1，-1。7行将情感词粘贴在一起。8行更改名称，为什么不用words这个词了呢?因为我觉得词典也用words命名很low，那样的话这几串代码看起来就差不多了。

同一个词可能有两种不同的情感倾向，尽管这更符合实际情况，但是还是在9行把它们去重，这里仅保留正向倾向。社会毕竟还是美好的嘛，传播正能量，构建和谐社会，从你我做起。duplicated函数的作用和unique函数比较相似，它返回重复项的位置编号，比如“爱”这个词在数据框和向量中第二次及更多次出现的位置，第一次出现的位置不返回，如果加了“非”函数，表示仅保留第一次出现的词汇。10、11、12行提取posneg的term列并使用Rwordseg包添加自定义词典的函数insertWords将词典添加入分词器。

到这里数据的准备就真的完成了，下面就可以进行分词。这里需要用到之前说过的Rwordseg分词包。

### 分词


```r
system.time(x <- segmentCN(strwords = sentence)) 
temp <- lapply(x, length)
temp <- unlist(temp)
id <- rep(test[, "id"], temp)
label <- rep(test[, "label"], temp)
term <- unlist(x)
testterm <- as.data.frame(cbind(id, term, label), stringsAsFactors = F)
```

1行使用分词函数segmentCN分词，其结果是一个和sentence等长的list，list的每一个元素是对应文本的分词结果。这里使用了system.time函数返回代码块的执行时间，因为文本处理的速度可能比较慢，预估一下出处理速度是个好习惯。当然我们数据量比较小，也就几秒中的事情。但是我们需要将分词结果和文本的id关联上，那文本分出多少个词就要重复多少次文本id，所以2行使用lapply函数返回x中每一个元素的长度,即文本分出多少个词；lapply返回的是一个list，所以3行unlist；然后4行将每一个对应的id复制相应的次数，就可以和词汇对应了；5行将id对应的情感倾向标签复制相同的次数；6行将list解散为向量；7行将一一对应的三个向量按列捆绑为数据框，分词整理就基本结束了。

在分析过程中，难免会产生很多中间变量，它们会占用大量内存，虽然dplyr包好像有个管道函数能够解决这个问题，但是它太小众了，用处不广。我们只要将临时中间变量命名为temp，保证在下一个temp出现之前，临时变量不会沿用就行了，这样临时变量就始终只有一个temp。

我记得上次有个什么烽火集团来找我们班的人做兼职，就是给文本添加标注，虽然我没去但是我真是觉得傻逼至及。人工标注啊，呵呵。不好意思我是个俗人，不说脏话表达不出我的情感，而且有些事情我实在是不想多说，和谐社会嘛。

下次碰到这种情况你们就可以去说：“你们打算花多少钱请人做兼职，我只要一半，全都给你们搞定。”只用复制我上面的几行代码就行了。

虽然算法已经足够简单，没有必要去除停用词，但是为了显示诚意，文本分析里每一个环节都不能少，这里还是认真的去除停用词，真的不是走过场哦。

### 去除停用词


```r
stopword<-read.csv("F:/study/新建文件夹/dict/stopword.csv",header=T,sep=",",stringsAsFactors=F)
stopword <- stopword[!stopword$term %in% posneg$term,]
testterm <- testterm[!testterm$term %in% stopword,]

```

读取停用词典；停用词中有可能有些词有感情色彩，既然要去除停用词，首先要将这类具有情感色彩的停用词从停用词表中去除。函数%in%在posneg$term中查找stopword的元素，如果查到了就返回真值，没查到就返回假，再加一个!将结果反向，除去停用词的任务也完成了。

## 情感指数计算

现在有了分词表testterm和情感词典posneg，另外已经给情感词添加了权重列，那么只需要匹配term列，即可给testterm表关联上情感权重，然后就可以计算篇文本的情感得分了。需要注意的是如果一个情感词出现多次，其权重加倍，去不去重是一个值得深思的问题，我觉得只要不出现一个操字打七八行的暴漫系评论，不去重更合理，当然你们也可以尝试一下去重后的结果会不会更好一点。


### 关联情感词权重


```r
library(plyr)
testterm <- join(testterm, posneg)
testterm <- testterm[!is.na(testterm$weight), ]
head(testterm)
#id term label weight
#1 10_zhangpig_2006-8-16_4.5.txt   的     1      1
#3 10_zhangpig_2006-8-16_4.5.txt   还     1      1
#4 10_zhangpig_2006-8-16_4.5.txt 没有     1     -1
#6 10_zhangpig_2006-8-16_4.5.txt   等     1      1
#7 10_zhangpig_2006-8-16_4.5.txt   了     1      1
#8 10_zhangpig_2006-8-16_4.5.txt   快     1      1
```

使用plyr包的join函数进行左关联，给testterm表关联上情感权重，然后筛除那些没有权重的非情感词汇。奇怪的是很多看似不具有情感的词，这些词典竟然也把他们当成了情感词，看来还需要优化情感词典。
  
### 计算情感指数


```r
dictresult<-aggregate(weight~id,data=testterm,sum)
dictlabel <- rep(-1, length(dictresult[, 1]))
dictlabel[dictresult$weight > 0] <- 1
dictresult <- as.data.frame(cbind(dictresult, dictlabel), stringsAsFactors = F)
```

使用透视表函数aggregate对weight列以文本id分组求和，就得出了每个文本的情感得分。（aggregate的用法在课本第115页，我就不说了）情感得分大于0，倾向为正，标记为1，反之标为-1；2行使用rep函数产生一个和dictresult等长的-1向量；3行筛选情感得分大于0的文本，然后根据筛选结果，将向量dictlabel相应的位置更改为1；将向量dictlabel和数据框dictresult捆绑在一起，dictlabel列即基于词典的情感分析法给出的文本标签。
  
## 方法评价：优缺点分析

词典法基本上省略了建模过程，也就用不着进行交叉检验了，只需要统计人工标记和词典标记的交叉表就可以反映出其准确性了。
  
### 交叉表评价


```r
temp<-unique(testterm[,c("id","label")])
dictresult <- join(dictresult, temp)
evalue <- table(dictresult$dictlabel, dictresult$label)
evalue
# -1    1
# -1  403   30
#  1  1561 1912
(403+1915)/(403+30+1561+1912)
#[1] 0.593446
```

筛选测试数据的文本id和人工标签label，并去重；2行temp和分类结果通过文本id左关联；table函数对人工标注列和词典标注列做交叉表。

结果有点崩啊，准确率只有59.3%，这就比瞎猜强一点。但从执行的过程中我们也发现，很多不具有情感色彩的词被定义为了情感词，例如的、了、还、在、我、都、把、上等字词，这些字词都是高频字词，而我们的计算方法按照出现频次重复计算，所以导致上面的结果偏差很大。

我们还是有很多可以优化的空间的，比如在词典里把例如的、了、还、在、我、都、把、上这些词去掉得到一个优化词典，或者上面说过的给每个情感词去重，只算它们出现一次的情况。我个人觉得细化情感权重是最好的方法，但是太困难了，对词典的要求太高了。

最后说一句，其实我对59.3%这个结果还是比较满意的。
