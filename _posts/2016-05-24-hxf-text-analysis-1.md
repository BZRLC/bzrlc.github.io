---
title: "文本分析初探-1"
author: hxf
date: "25 May 2016"
layout: single
header: "mm-layout-examples.png"
comments: true
categories: [NLP]
tags: [NLP,R]
mathjax: false
highlight: true
description: "第一次投稿关于文本处理与分析方面的文章，请轻拍 :pray: "
---


## 开始 
为什么要从文本分析开始，其实我并不打算以后做这个。但人总是会做矛盾的事情，我相当缺乏去主攻一个方向的自信，只是抱着不在一颗树上吊死的念头随便看了看。张志刚老师建议我多去研究量化投资方面的内容，我以后会去多看看这个方面，我现在只是知道有这么个东西而已，有想法和我一起尝试的人我非常欢迎。

言归正转，为什么我要从文本分析开始整理，因为我觉得它逼格赛高，讲文本分析的资料非常少，这样来和我撕逼的人应该也比会少了
。
我手上有一份从京东，当当，和携程网上爬去下来的一些用户的评论。当然不是我爬的，是我用百度网盘下载的，其实我也不是不会用爬虫……好吧，我就是不会，反正我是没有办法爬的这么干净的，有机会爬虫我也可以给大家试一下。

首先我们看看怎么读入文件(除非特别声明，我使用的软件都是R)。

{% highlight r %}
#获取文本路径
reviewpath<-("F:/study/新建文件夹/rawdata/review_sentiment/train2")

completepath <- list.files(reviewpath, pattern = "*.txt$", full.names = TRUE)

# 注意是左斜杠,但是也是可以用两个反斜杠的 "\\"
reviewpath <- ("F:/study/新建文件夹/rawdata/review_sentiment/train2")
{% endhighlight %}

list.files函数获取该路径下所有文件的文件名，如果full.names参数为真，则返回完整的路径，反之返回文件名称，pattern是一个匹配函数，用来匹配指定的文件（更多有关匹配原理和规则请自行参考正则表达式），这里仅匹配以.txt结尾的文件。


## 批量读入文本

{% highlight r %}

read.txt <- function(x){
  des <- readLines(x)
  return(paste0(des, collapse = "")
  }
  
review <- lapply(completepath, read.txt)

{% endhighlight %}
 
这里为批量读取文件构造了一个函数，目的是把每一个TXT文件的目录加载进来并粘贴在一起。第一行代码构造了一个自编函数，第二行<code>readLines</code>函数逐行读取文件目录，第三行将目录以空格粘贴在一起，<code>collapse</code>就是连接方式参数。最后用lapply函数按目录逐行读取文件(appply系列的函数可能一开始很难理解，我会把资料一起上传的)。

如果你想编一个循环来代替这个步骤，那你真的是LOW爆了，不到万不得已R里不要用循环，多用向量化的思维。最后一行的报错我也不知道是什么意思，反正到最后都没什么影响。


{% highlight r %}

#读取文件名称
docname<-list.files(reviewpath,pattern="*.txt$")

#名称和文本内容按列（cbind）捆绑在一起成为一个新的数据框
reviewdf <- as.data.frame(cbind(docname, unlist(review))  , stringsAsFactors = F) 

#重命名列名
colnames(reviewdf) <- c("id", "words")

#正则替换掉空格
reviewdf$words <- gsub(pattern = "\\s", replacement ="", reviewdf$words)

#正则去掉标点符号
reviewdf$words <- gsub(pattern = "[[:punct:]]", replacement = "", reviewdf$words)

{% endhighlight %}

csv各式的文档以英文逗号为分隔符，文中有英文逗号会报错，除了英文逗号可能引起<code>read.csv</code>函数读取csv文件报错以外，还有英文单引号、英文双引号、波浪号（~），都会引起读取时发生警告，带来csv文件或txt文件读取不完整的后果.

> 小赵补充说明一下

{% highlight r %}

> test <- readline() #输完下面然后enter
"英文双引号" '英文单引号' ~~ 英文逗号, 中文逗号，over。.

#带有英文双引号的会被加反斜杠转义
> test
[1] "\"英文双引号\" '英文单引号' ~~ 英文逗号, 中文逗号，over。."

> gsub("[[:punct:]]","",test)
[1] "英文双引号 英文单引号  英文逗号 中文逗号over"

#同时用[]包括进去感觉要比"|"或这个更方便一点
> gsub("[~,']", "",test)
[1] "\"英文双引号\" 英文单引号  英文逗号 中文逗号，over。."

#去掉空格，正则是\s 在匹配的时候要加反斜杠
> gsub("\\s", "",test)
[1] "\"英文双引号\"'英文单引号'~~英文逗号,中文逗号，over。."

#去掉英文引号可以直接用\" 转义一遍就行了。
> gsub("\"", "",test)
[1] "英文双引号 '英文单引号' ~~ 英文逗号, 中文逗号，over。."

{% endhighlight %}

更多关于R的正则看[这里](http://blog.163.com/yugao1986@126/blog/static/692285082014324402546/)

到此为止，数据的前期处理部分算是完成了，预知后事如何，请听下回分解。

