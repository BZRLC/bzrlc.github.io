---
title: "获取 R 函数或方法源代码"
author: "Lix"
date: "10 November 2016"
header:
    overlay_color: "#ccc"
    overlay_image: share-img.jpg
    padding: 9.5em
excerpt: "学编程要多阅读源代码，在此整理从 R 命令行获取源代码的方法。"
layout: single
comments: true
categories: [R-Basics]
tags: [R,Programming]
highlight: true
mathjax: false
description: "学编程要多阅读源代码，在此整理从 R 命令行获取源代码的方法。"
---

为了深入了解 R 语言，整理了一些阅读源代码的方法，在此整理和记录以便查阅。

要直接从 R 命令行获取源代码，需要了解一些 R 语言编程知识。R 语言中函数或者方法有许多种，例如：

- in-built 函数
- S3 范型函数
- S4 范型函数
- non-visible functions（隐藏函数）
- unexported functions
- compiled code
- infix operators

不同类型函数获取源代码的方式不同。要获取源代码，首先直接执行函数（不带括号），观察返回结果。一般来说，普通函数会直接返回源代码。

如果返回的结果包含 `UseMethod("function")`，则表示该函数为 S3 范型函数，它根据对象类型的不同调用不同的方法。然后使用 `methods()` 列出特定的范型函数和与类有关的函数 `methods(function)` `methods(class="classname")`。我们可以看到返回一些带 `*` 的函数，这类函数为 `Non-visible functions`，即隐藏的函数，表示这类函数没有从语言包的命名空间导出。但是我们仍然可以查看隐藏函数的源代码，可通过 `:::` 函数查看，即 `pkgname:::function`。另外，还可以使用 `getAnywhere()` 获取。`getAnywhere()` 并不需要知道函数来自于哪个语言包。对于没有 `*` 的方法，直接运行即可获得源代码。另外，S3 范型函数还可以通过 `getS3method(function, class)` 获取源代码。

如果返回的结果包含 `standardGeneric("function")`，意味着该函数为 S4 范型函数。对于这类函数，先通过 `showMethods(function)` 获得方法对应的 `signature`（不知道是神马东西）。例如：

``` r
> showMethods(chol2inv)
Function: chol2inv (package base)
x="ANY"
x="CHMfactor"
x="denseMatrix"
x="diagonalMatrix"
x="dtrMatrix"
x="sparseMatrix"
```

其中 `x="ANY"` 就是 signature。那么，可以通过 `getMethod("function", "signature")` 获取源代码。
有一些方法具有更复杂的 `signature`，例如：

``` r
require(raster)
showMethods(extract)
Function: extract (package raster)
x="Raster", y="data.frame"
x="Raster", y="Extent"
x="Raster", y="matrix"
x="Raster", y="SpatialLines"
x="Raster", y="SpatialPoints"
x="Raster", y="SpatialPolygons"
x="Raster", y="vector"
```

那么，可以通过 `getMethod("function", signature = c(x = "Raster", y = "SpatialPolygons"))` 获取源代码。

对于 `unexported functions`，可以使用和隐藏函数一样的方法获取源代码，即 `:::` 或 `getAnywhere()`。对于 `compiled code`，即返回内容中有 `<bytecode:0x294e410>` 的函数，仍然可以从 R 命令行查阅源代码。但是对于函数中调用的 `.C` `.Call` `.Fortran` `.External` `.Internal` `.Primitive`，无法从命令行获取源代码，需要直接查看编译前的源代码。函数 `pryr::show_c_source` 可以直接在 Github 中找到 `.Internal` 和 `.Primitive` 调用的内容。

对于无法直接从命令行获取源代码的函数，可以直接下载原始语言包。

``` r
## from CRAN
download.packages(pkgs = "Matrix",
                  destdir = ".",
                  type = "source")
## uncompressing and untaring
untar(download.packages(pkgs = "Matrix",
                        destdir = ".",
                        type = "source")[,2])
```

基本包的源代码，可以通过查阅 R 语言源代码： [Subversion repository](http://svn.r-project.org/R/trunk/) 或者 [Winston Chang's github mirror](https://github.com/wch/r-source/tree/trunk)。对于其他来源的包，直接从相应网站获取。

最后，可以通过 `edit(getAnywhere("function"), file = "source_function.r")` 或者 `capture.output(getAnywhere("function"), file = "source_function.r")` 获得源代码的文本内容。

另外，还可以通过代码调试方法获得源代码 `debugonce(function)`。使用 `debug(function)` 需要 `undebug()` 结束调试，相对麻烦一点。

对于 infix operators，即 `%%` `%*%` `%in%`，可以通过 `getAnywhere()` 或者使用 backticks (`) 包裹然后执行即可返回源代码。

---

说明：该文章同时发布在作者[个人博客](http://lix90.github.io)上，内容主要从网络上获取整理，主要参考了这个 [stackoverflow](http://stackoverflow.com/questions/19226816/how-can-i-view-the-source-code-for-a-function) 问题的解答。未经过专业编程训练，措辞难免存在不恰当之处，还请指正。

主要参考资料：

- [How can I view the source code for a function?](http://stackoverflow.com/questions/19226816/how-can-i-view-the-source-code-for-a-function) 主要参考资料
- [What does “S3 methods” mean in R?](http://stackoverflow.com/questions/6583265/what-does-s3-methods-mean-in-r/6583639#6583639)
- [Get source codes for invisible functions or internal functions in R](http://yusung.blogspot.jp/2007/08/get-invisible-functions-or-internal.html)
- [How do I show the source code of an S4 function in a package?](http://stackoverflow.com/questions/5937832/how-do-i-show-the-source-code-of-an-s4-function-in-a-package)
