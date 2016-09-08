---
layout: single
title: "探索性数据分析"
author: "Elle Liu"
date: "9 September 2016"
categories: [Description-Statistics]
tags: [Data-Explore]
mathjax: ture
highlight: ture
description: "介绍一整套的数据探索，描述，到ggplot作图流程"
---

## 1.探索数据结构

我们可以利用`str()`来观察数据集的结构。

{% highlight r %}
usedcars<-read.csv("usedcars.csv",header=T,stringsAsFactors = F)
str(usedcars)

## 'data.frame':	150 obs. of  6 variables:
##  $ year        : int  2011 2011 2011 2011 2012 2010 2011 2010 2011 2010 ...
##  $ model       : chr  "SEL" "SEL" "SEL" "SEL" ...
##  $ price       : int  21992 20995 19995 17809 17500 17495 17000 16995 16995 16995 ...
##  $ mileage     : int  7413 10926 7351 11613 8367 25125 27393 21026 32655 36116 ...
##  $ color       : chr  "Yellow" "Gray" "Silver" "Gray" ...
##  $ transmission: chr  "AUTO" "AUTO" "AUTO" "AUTO" ...
{% endhighlight %}

`usedcars`数据集的变量较少，只有6个，当数据集变量很多时，可以采用一些简单的命令来进行观察。
当我们想知道每个变量所对应的数据类型,则有：

{% highlight r %}
(s<-sapply(usedcars,class))

##         year        model        price      mileage        color 
##    "integer"  "character"    "integer"    "integer"  "character" 
## transmission 
##  "character"

table(s)

## s
## character   integer 
##         3         3
{% endhighlight %}

当我们想提取不同的数据类型进行分别分析，则有：


{% highlight r %}
int<- s %in% "integer"
usedcars.int<-usedcars[,int]
knitr::kable(head(usedcars.int))
{% endhighlight %}


| year| price| mileage|
|:---:|:----:|:------:|
| 2011| 21992|    7413|
| 2011| 20995|   10926|
| 2011| 19995|    7351|
| 2011| 17809|   11613|
| 2012| 17500|    8367|
| 2010| 17495|   25125|

此时得到的`usedcars.int`就是一个仅包含int类型变量的数据集。

## 2.探索数值型变量

对于数值型数据的探索我们常使用汇总统计量。
    

{% highlight r %}
summary(usedcars.int)

##       year          price          mileage      
##  Min.   :2000   Min.   : 3800   Min.   :  4867  
##  1st Qu.:2008   1st Qu.:10995   1st Qu.: 27200  
##  Median :2009   Median :13592   Median : 36385  
##  Mean   :2009   Mean   :12962   Mean   : 44261  
##  3rd Qu.:2010   3rd Qu.:14904   3rd Qu.: 55125  
##  Max.   :2012   Max.   :21992   Max.   :151479
{% endhighlight %}

可以看到`summary()`主要为我们提供了数据两方面的信息，一方面是数据的**中心趋势**（均值、中位数），另一方面是数据**分散程度**（分位数）。

### 2.1中心趋势
    
对于`price`可以看到它的均值和中位数非常相似，相差大约5.2%(`13592/12926=1.052`),但是`mileage`的均值比中位数大了约21.6%(`44261/36385=1.216`)。我们知道均值的稳健性差，而`mileage`均值大于中位数，说明它存在极大值，这一点在后面可视化部门将更为直观。
    由于均值容易受到极端值的影响，因此我们有时会采用**截尾均值**来测度中心趋势。
    

{% highlight r %}
mean(usedcars$price,trim=0.25) #数据排序后，删去左右各25%的数据，再求均值

## [1] 13373.62
{% endhighlight %}

### 2.2分散程度

`range()`函数能够同时返回最小值和最大值，将它与`diff()`结合使用可以得到数据的全距。
    

{% highlight r %}
range(usedcars$price)

## [1]  3800 21992
diff(range(usedcars$price))

## [1] 18192
{% endhighlight %}

一般来说我们对于数据第一四分位数Q1和第三四分位数Q3之间50%的数据比较感兴趣，因为它们就是数据分散程度的一个测度，Q1和Q3之差我们称为四分位距IQR。同时我们也可以用`quantile()`来得到我们想要的任意分位数。
    

{% highlight r %}
IQR(usedcars$mileage)

## [1] 27924.25

quantile(usedcars$mileage)

##        0%       25%       50%       75%      100% 
##   4867.00  27200.25  36385.00  55124.50 151479.00

quantile(usedcars$mileage,probs = seq(from=0,to=1,by=0.20))

##       0%      20%      40%      60%      80%     100% 
##   4867.0  24017.4  34708.8  40106.8  63951.8 151479.0
{% endhighlight %}

`mileage`的分散程度显示出Q3和max之差(`151479-55125=96354`)远远大于Q1和min之差(`27200-4867=22333`),即较大的值比较小的值更为分散。

### 2.3数据可视化

利用箱线图、直方图和密度图能够帮助我们直观的了解到数据的分布形态。
    

{% highlight r %}
library(ggplot2)
price1<-ggplot(usedcars,aes(x=1,y=price))+geom_boxplot()+
        scale_x_continuous(breaks = NULL)+    # 移走刻度
        xlab("price")  
#       theme(axis.title.x=element_blank())  # 移走x轴标签
price1
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/images/posts/Data-Explore/unnamed-chunk-8-1.png)

{% highlight r %}
mileage1<-ggplot(usedcars,aes(x=1,y=mileage))+geom_boxplot()+
        scale_x_continuous(breaks = NULL)+xlab("mileage")  +  
        stat_summary(fun.y = "mean",geom="point",shape=23,fill="white")  #向箱线图添加均值
mileage1
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/images/posts/Data-Explore/unnamed-chunk-8-2.png)
    
可以看到`mileage`中的均值与中位数相隔较远，而且在较大的数据中存在比较多的异常值，与之前的分析一致。


{% highlight r %}
library(ggplot2)
price2<-ggplot(usedcars,aes(x=price,y=..density..))+
   geom_histogram(fill="cornsilk",colour="grey60",size=.2)+
   geom_line(stat = "density")+
   xlim(3800,22000)
price2
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/images/posts/Data-Explore/unnamed-chunk-9-1.png)

{% highlight r %}
mileage2<-ggplot(usedcars,aes(x=mileage,y=..density..))+
  geom_histogram(fill="cornsilk",colour="grey60",size=.2)+
  geom_line(stat = "density")+
  xlim(4800,155000)
mileage2
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/images/posts/Data-Explore/unnamed-chunk-9-2.png)

可以看到`mileage`呈现明显的偏态分布.
    
## 3.探索分类变量

首先我们可以得到一个仅包含分类变量的数据集`usedcars.cha`
    

{% highlight r %}
usedcars.cha<-usedcars[,!int]
str(usedcars.cha)

## 'data.frame':	150 obs. of  3 variables:
##  $ model       : chr  "SEL" "SEL" "SEL" "SEL" ...
##  $ color       : chr  "Yellow" "Gray" "Silver" "Gray" ...
##  $ transmission: chr  "AUTO" "AUTO" "AUTO" "AUTO" ...
{% endhighlight %}

例如我们想知道对于成交的二手车，什么颜色数量最多，则有：

{% highlight r %}
sort(table(usedcars$color))
## 
##   Gold Yellow  Green   Gray  White   Blue    Red Silver  Black 
##      1      3      5     16     16     17     25     32     35
{% endhighlight %}

有时直接给出占比会更容易理解，则有：
    

{% highlight r %}
prop.table(table(usedcars$color))*100

## 
##      Black       Blue       Gold       Gray      Green        Red 
## 23.3333333 11.3333333  0.6666667 10.6666667  3.3333333 16.6666667 
##     Silver      White     Yellow 
## 21.3333333 10.6666667  2.0000000
{% endhighlight %}

可以直观的看到二手车中，黑色车最普遍，约为23.33%，其次是银色车，有21.33%。

## 4.探索变量之间的关系

对于数值型变量我们常用`plot()`来散点图，揭示`x`变量和`y`变量之间的关系,研究当`x`增加时，`y`是如何变化的。
对于名义变量之间的关系，我们常用**双向交叉表**来观察一个变量的值是如何随着另外一个值得变化而变化的。
例如对于变量`model`和`color`，我们想探讨不同车型与颜色是否具有相关性。我们运用gmodels包中的`CrossTable()`函数来进行分析。
再进一步分析之前，我们可以减少`color`变量的个数来简化任务，因为它有9个不同水平，但我们只想知道颜色保守与否对车型是否有关。因此，我们将颜色分成两组：
    

{% highlight r %}
usedcars$conservative<-
  usedcars$color %in% c("Black","Silver","White","Gray")
{% endhighlight %}

`%in%`操作符，它根据左边的值是否在右边的向量中，为左边向量的每个值返回TRUE或者FALSE。
观察一下我们的指示变量`conservative`,可知有2/3的汽车有保守的颜色。
    

{% highlight r %}
table(usedcars$conservative)
## 
## FALSE  TRUE 
##    51    99
{% endhighlight %}

因为我们假设汽车型号决定了颜色的选择，所以我们把`conservative`作为因变量(`y`),CrossTable()应用如下：
    

{% highlight r %}
library(gmodels)
CrossTable(x=usedcars$model,y=usedcars$conservative,chisq = TRUE)
{% endhighlight %}



{% highlight text %}
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## | Chi-square contribution |
## |           N / Row Total |
## |           N / Col Total |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  150 
## 
##  
##                | usedcars$conservative 
## usedcars$model |     FALSE |      TRUE | Row Total | 
## ---------------|-----------|-----------|-----------|
##             SE |        27 |        51 |        78 | 
##                |     0.009 |     0.004 |           | 
##                |     0.346 |     0.654 |     0.520 | 
##                |     0.529 |     0.515 |           | 
##                |     0.180 |     0.340 |           | 
## ---------------|-----------|-----------|-----------|
##            SEL |         7 |        16 |        23 | 
##                |     0.086 |     0.044 |           | 
##                |     0.304 |     0.696 |     0.153 | 
##                |     0.137 |     0.162 |           | 
##                |     0.047 |     0.107 |           | 
## ---------------|-----------|-----------|-----------|
##            SES |        17 |        32 |        49 | 
##                |     0.007 |     0.004 |           | 
##                |     0.347 |     0.653 |     0.327 | 
##                |     0.333 |     0.323 |           | 
##                |     0.113 |     0.213 |           | 
## ---------------|-----------|-----------|-----------|
##   Column Total |        51 |        99 |       150 | 
##                |     0.340 |     0.660 |           | 
## ---------------|-----------|-----------|-----------|
## 
##  
## Statistics for All Table Factors
## 
## 
## Pearson's Chi-squared test 
## ------------------------------------------------------------
## Chi^2 =  0.1539564     d.f. =  2     p =  0.92591 
## 
## 
## 
{% endhighlight %}

CrossTable()的显示结果中包含了大量数据，最上面一张表显示出单元格的值该如何解释。我们感兴趣的是保守型的颜色在不同车型中的行占比，`0.654`的`SE`车型用保守颜色，而`SEL`车型保守颜色的比例是`0.696`,`SES`车型则是`0.653`，这些数据的差异是比较小的，因此可以认为不同类型的汽车对于颜色选择没有显著差异。

最后的卡方独立性检验，其原假设为认为变量不相关，备择假设是变量相关，此处我们的`p-value=0.926`,不能拒绝原假设，因此认为单元的数量变化可能是由于偶然，而不是因为`model`和`color`真的存在相关关系。

