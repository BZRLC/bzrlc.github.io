---
title: "R 语言基本数据类型"
author: "Lix"
date: "25 November 2016"
header:
    overlay_color: "#ccc"
    overlay_image: r-basics.png
    padding: 10em
excerpt: "了解下 R 语言基本数据类型的创建和操作。"
layout: single
comments: true
categories: [R-Basics]
tags: [R,Data-type]
highlight: true
mathjax: false
description: "了解下 R 语言基本数据类型的创建和操作。"
---

# Vector 向量

向量 vector 由同类型的元素构成。

## 向量创建

`vector` 创建给定长度和模式的向量。

``` r
>>> vector(mode = "character", length = 5)
[1] "" "" "" "" ""
```

`as.vector` 可以将实参转换为给定模式的向量。如果输入结果为原子模式（atomic），属性将丢失。R 中原子模式为 "logical", "integer", "numeric" ("double"), "complex", "character", 和 "raw"。在 R 中原子模式应该就是指最基本的模式，能够组合成其他的复杂模式，例如 "list" "expression"。

``` r
>>> x <- c(a = 1, b = 2)
a b
1 2
>>> is.vector(x)
[1] TRUE
>>> as.vector(x)
[1] 1 2
all.equal(x, as.vector(x))
[1] "names for target but not for current"
```

`c` 可以将它的实参组合成一个向量。所有的实参都将被强制转换为通用的类型，除了 names 之外，其他属性都将丢失。输出数据类型由实参中元素的最高级类型决定（NULL < raw < logical < integer < double < complex < character < list < expression）。

例如：

转换为字符串。

``` r
>>> (out <- c(1, "2"))
[1] "1" "2"
```

转换为 list

``` r
>>> (out <- c(1, list(2)))
[[1]]
[1] 1

[[2]]
[1] 2
```

### 向量运算

**向量的索引**

- `a[1]` 单个元素的索引
- `a[1:2]` `a[c(1,3)]` 多个元素的索引
- `a[a==5]` 使用逻辑值来索引
- `a[-1]` `a[c(-1,-3)]` 排除元素

**向量的运算**

- `a*2` 向量与2相乘，2将与向量的每一个元素相乘。
- `a*b` 向量与向量相乘，对应位置的元素相乘。如果 `a` 和 `b` 的长度不等长呢？试一个例子：

``` r
>>> v1 <- c(1,2)
>>> v2 <- c(3,3,3)
>>> v3 <- c(4,4,4,4)
>>> v1*v2
[1] 3 6 3
Warning message:
In v1 * v2 : 长的对象长度不是短的对象长度的整倍数
>>> v1*v3
[1] 4 8 4 8
```

也就是说，向量相乘时，若长度不相等，较短的向量会继续往后与长的向量相乘。如果向量的长度不是整数倍，会出现警告：“长的对象长度不是短的对象长度的整倍数”。

- `a%%2` 对向量所有元素求余数
- `a%/%2.4` 向量所有元素与2.4进行整除运算
- `t(a)` 对 `a` 进行转置

**总结：对于向量的运算，都是对其元素的运算。**

## Matrix 矩阵（以及数组 array）

### 矩阵创建

`matrix(data = NA, nrow = 1, ncol = 1, byrow = FALSE, dimnames = NULL)`

- `data` 为向量类型，非原子模式的对象将由 `as.vector` 强制转换为向量，并且去掉属性。
- `nrow` `ncol` 指定矩阵的行或列的数目，如果两个参数均未指定，那么将得到一个单列矩阵。
- `byrow` 逻辑值，若 `FALSE`（默认），矩阵按列来填充；若 `TRUE` 则按行填充。
- `dimnames` 矩阵的名称属性，`NULL` 或者长度为 list 的对象，空 list 为 `NULL`。list 的第一个元素的值被当作矩阵行的名称。list 也可以含有名称，那么 list 的名称将被当作矩阵的维度名称。

``` r
>>> mdat <- matrix(c(1,2,3,11,12,13), nrow = 2, ncol = 3, byrow = TRUE,
+ dimnames = list(c("R.1", "R.2"), c("C.1", "C.2", "C.3")))
>>> mdat
     C.1 C.2 C.3
row1   1   2   3
row2  11  12  13
```

### 矩阵运算

- `a[,2]` `a[1,2]` `a[c(1,2),]` `a[-2,]` 矩阵索引或者取子矩阵。
- `rbind(a,b)` `cbind(a,c)` 将两个矩阵或向量按照行或者列合并。
- `a*b` 一对一乘积（点积），行和列数必须匹配。
- `a%*%b` 矩阵乘积，`a` 的列必须等于 `b` 的行。

``` r
>>> a <- matrix(1:6, nrow = 2)
>>> b <- matrix(2:7, nrow = 3)
>>> a*b
Error in a * b : 非整合陈列
>>> a*t(b)
[,1] [,2] [,3]
[1,]    2    9   20
[2,]   10   24   42
>>> a%*%b
[,1] [,2]
[1,]   31   58
[2,]   40   76
>>> a%*%t(b)
Error in a %*% t(b) : 非整合参数
```

`apply(a, MARGIN, FUN, ...)` 对矩阵或数组的向量化运算，如果存在名称的话，`MARGIN` 也可为维度的名称字符串。在此只讨论对矩阵的用法。

- `apply(a, MARGIN = 1, sum)` 对矩阵 a 的行求和。
- `apply(a, MARGIN = 2, sum)` 对矩阵 a 的列求和。
- `apply(a, MARGIN = C(1,2), sum)` 对矩阵 a 的行列求和，得到的仍为原来的矩阵。

``` r
>>> a <- matrix(1:6, nrow = 2)
>>> apply(a, 1, sum)
[1]  9 12
>>> apply(a, 2, sum)
[1]  3  7 11
>>> apply(a, c(1,2), sum)
     [,1] [,2] [,3]
[1,]    1    3    5
[2,]    2    4    6
```

- `diag(a)` 提取矩阵 a 的对角元素。
- `diag(1:4)` 构建一个新的对角矩阵。
- `crossprod(a,b)` 矩阵叉积，等同于 `t(a)%*%b`，前者计算效率更高。

### Array 数组创建

`array(data = NA, dim = length(data), dimnames = NULL)`

数组可以存储两个以上维度数据。矩阵其实是特殊的两个维度的数组。`dim` 用于指定每个维度的长度。如果 `data` 数据长度不足，将会被复制循环填充。

``` r
>>> arr <- array(1:12, dim = c(2,2,3))
>>> arr
, , 1

[,1] [,2]
[1,]    1    3
[2,]    2    4

, , 2

[,1] [,2]
[1,]    5    7
[2,]    6    8

, , 3

[,1] [,2]
[1,]    9   11
[2,]   10   12
```

## factor 因子

因子是一个由字符串或者整数组成的向量，用来对另外一个等长的向量进行分类的离散变量。与一般的向量不同点是 factor 具有 `level` 属性。R 提供了已排序和未排序两类 factor。

``` r
factor(x = character(), levels, labels = levels,
    exclude = NA, ordered = is.ordered(x), nmax = NA)
```

- `x` 向量，一般为少数几个不同的值。向量 `x` 的类型必须可以被转换为字符串 `as.character()` 和可以被排序 `sort.list`。
- `levels` 水平，`x` 中选择性的值构成的字符串向量。默认值为 `unique(as.character(x))`，并且增序排列。`levels` 可以少于 `sort(unique(x))`。如果 `levels` 少于 `sort(unique(x))`，`x` 中没有包含的元素将被当作 `NA`。
- `labels` 可选的参数，用来命名因子水平名称。例如，如果 `labels = "f"`，那么因子名为 `f1, f2, ...`。
- `exclude` 在组成 `levels` 时需要排除掉的元素。与 `x` 类型相同，否则需要强制转换。
- `ordered` 是否有序。有序的因子与未排序的因子仅仅在类上不同，但是方法和模型拟合函数对两者的处理有非常大的不同。
- `nmax` 水平数的上限。

`factor` 函数返回一个 "factor" 类的对象，这个对象具有和 `x` 等长的整数码，具有 "character" 模式的 "levels" 属性。如果 `ordered = TRUE` 或者使用了 `ordered()` ，那么对象则具有两个类 `c("ordered", "factor")`。对因子的解释依赖于编码（codes）和水平属性（levels）两个因素，所以在对因子进行比较时需谨慎。对因子使用 `as.numeric` 没有意义，因为会自动强制转换。如果要将因子 `f` 转换为初始的数值，建议使用 `as.numeric(levels(f))[f]`，这样要比 `as.numeric(as.character(f))` 效率高。因子的水平默认进行了排序，但是排序的标准依赖本地区域设置。可能不是基于 ASCII。尽量不要使用 `NA` 作为水平值。

### 因子的创建和操作

``` r
>>> a <- c(1,1,2,2)
>>> f <- factor(a)
>>> f
[1] 1 1 2 2
Levels: 1 2

## 指派了 levels 结果一致
>>> f <- factor(a, levels = c("1", "2"))
>>> f
[1] 1 1 2 2
Levels: 1 2

## levels 个数少于 a
>>> f <- factor(a, levels = c("1"))
>>> f
[1] 1    1    <NA> <NA>
Levels: 1

## 增加标签
>>> f <- factor(a, labels = "x")
>>> f
[1] x1 x1 x2 x2
Levels: x1 x2

## 增加排序
>>> f <- factor(a, labels = "x", ordered = TRUE)
>>> f
[1] x1 x1 x2 x2
Levels: x1 < x2

## 改变 levels 的顺序
>>> f <- factor(a, levels = c("2", "1"), labels = "x", ordered = TRUE)
>>> f
[1] x2 x2 x1 x1
Levels: x1 < x2
```

``` r
## 构建一个 data.frame
>>> c.1 <- rep(c("f1", "f2"), each = 3)
>>> c.2 <- 1:6
>>> d <- data.frame(c.1, c.2)
>>> d
  c.1 c.2
1  f1   1
2  f1   2
3  f1   3
4  f2   4
5  f2   5
6  f2   6

## 将 c.1 转换为因子
>>> (d$c.1 <- factor(c.1))
[1] f1 f1 f1 f2 f2 f2
Levels: f1 f2

## 或者使用 as.factor
>>> as.factor(d$c.1)
[1] f1 f1 f1 f2 f2 f2
Levels: f1 f2

## 进行一个排序
>>> factor(d$c.1, levels = c("f2", "f1"), ordered = TRUE)
[1] f1 f1 f1 f2 f2 f2
Levels: f2 < f1

>>> factor(d$c.1, levels = c("f1", "f2"), ordered = TRUE)
[1] f1 f1 f1 f2 f2 f2
Levels: f1 < f2

## 或者使用 ordered
>>> ordered(d$c.1, levels = c("f1", "f2"))
[1] f1 f1 f1 f2 f2 f2
Levels: f1 < f2

>>> ordered(d$c.1, levels = c("f2", "f1"))
[1] f1 f1 f1 f2 f2 f2
Levels: f2 < f1

## 转换因子的数据模式
>>> as.character(d$c.1)
[1] "f1" "f1" "f1" "f2" "f2" "f2"

>>> as.numeric(d$c.1)
[1] 1 1 1 2 2 2

>>> as.logical(d$c.1)
[1] NA NA NA NA NA NA
```

# List 列表

list 是 R 中比较宽松的数据类型，它可以由类型不一致的任意对象构成。list 非常适合用于封装函数的输出对象。list 的元素可以使用 `$` `[]` `[[]]` 访问。

`l = list(tag1 = value1, tag2 = value2, ..., tagn = valuen)`

``` r
>>> x = list(a = 1:10, beta = exp(-3:3),
+ logic = c(TRUE,FALSE,FALSE,TRUE))

>>> x
$a
[1]  1  2  3  4  5  6  7  8  9 10

$beta
[1]  0.04978707  0.13533528  0.36787944  1.00000000  2.71828183  7.38905610
[7] 20.08553692

$logic
[1]  TRUE FALSE FALSE  TRUE

>>> x$a
[1]  1  2  3  4  5  6  7  8  9 10
>>> x[1]
$a
[1]  1  2  3  4  5  6  7  8  9 10

>>> x[[1]]
[1]  1  2  3  4  5  6  7  8  9 10
>>> x[[1]][1]
[1] 1
>>>
```

`apply` 函数簇中 `lapply` 函数可以对 list 进行向量化运算，返回的数据类型也是 list。`sapply` 同样支持 list 类型的参数，但是默认返回的数据类型为 vector。但是当传入参数 `simplify = FALSE`，返回 list。

``` r
>>> lapply(x, mean)
$a
[1] 5.5

$beta
[1] 4.535125

$logic
[1] 0.5

>>> sapply(x, mean)
       a     beta    logic
5.500000 4.535125 0.500000

>>> class(lapply(x,mean))
[1] "list"

>>> class(sapply(x,mean))
[1] "numeric"

>>> is.matrix(sapply(x,mean))
[1] FALSE

>>> is.vector(sapply(x,mean))
[1] TRUE

>>> is.list(sapply(x,mean,simplify = FALSE))
[1] TRUE
```

其他与 list 有关的函数

`unlist(x, recursive = TRUE, use.names = TRUE)` 将 list 展开，转换为 vector

``` r
>>> unlist(x)
         a1          a2          a3          a4          a5          a6
 1.00000000  2.00000000  3.00000000  4.00000000  5.00000000  6.00000000
         a7          a8          a9         a10       beta1       beta2
 7.00000000  8.00000000  9.00000000 10.00000000  0.04978707  0.13533528
      beta3       beta4       beta5       beta6       beta7      logic1
 0.36787944  1.00000000  2.71828183  7.38905610 20.08553692  1.00000000
     logic2      logic3      logic4
 0.00000000  0.00000000  1.00000000
```

`as.list(x, all.names = FALSE, sorted = FALSE, ...)` 将对象数据类型转换为 list
`is.list(x)` 检测数据类型是否为 list

# Data.frame 数据框

data.frame 是 R 中非常重要的数据类型，它长得像 matrix，但是又与 list 一样，可以存储不同类型的数据，但是有一个每列数据长度必须一致。同样，data.frame 数据的访问与 matrix 和 list 类似。

``` r
data.frame(..., row.names = NULL, check.rows = FALSE,
           check.names = TRUE,
           stringsAsFactors = default.stringsAsFactors())
```

参数

- `...` 参数形式为 value 或者 tag = value，value 为数据内容，tag 为列名。
- `row.names` 行的名称
- `check.rows` 是否检测行数和名称是否一致。
- `check.names` 是否检测列的名称的合法性以及是否重复，有必要的话，通过 `make.names` 调整名称。
- `stringsAsFactors` 是否将字符向量转换为因子类型。如果不想让某个对象被强行转换类型，可以用 `I(a)` 包裹。

``` r
>>> d <- data.frame(c1 = rep(c("f1", "f2"), each = 3), c2 = 1:6)
>>> d
  c1 c2
1 f1  1
2 f1  2
3 f1  3
4 f2  4
5 f2  5
6 f2  6

>>> d$c1
[1] f1 f1 f1 f2 f2 f2
Levels: f1 f2

>>> d[1]
c1
1 f1
2 f1
3 f1
4 f2
5 f2
6 f2

>>> d[1,2]
[1] 1

>>> d["c1"]
c1
1 f1
2 f1
3 f1
4 f2
5 f2
6 f2

>>> d["c1"][1,]
[1] f1
Levels: f1 f2
```

此博文同时在作者[个人博客](http://lix90.github.io)发布。菜鸟笔记，如有错误，欢迎指正。
