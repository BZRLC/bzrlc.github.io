
This blog is forked form [minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/), a flexible two-column Jekyll theme. Perfect for personal sites, blogs, and portfolios hosted on GitHub or your own server

## 说明

不知道大家有没有碰到这样的问题：之前遇到了的问题再次遇到时却记不起当时的解决办法了，或者记录在了某张纸上/某个本子上，想去翻却找不到的尴尬  :flushed:

所以我建了这个博客项目，来详细记录每次碰到的R中各种问题和解决方法以及感悟。一来，可以作为备忘，当你忘记的时候，只要有网就可以打开<http://bzrlc.tech>来看看自己写的东西；二来，作为一个交流平台，大家可以互相学习、交流，共同进步！Together, we make progress!

博客的所有文章均遵循[CC BY-NC-SA 3.0 CN 协议](https://creativecommons.org/licenses/by-nc-sa/3.0/cn/)，分享、演绎需准守以下原则:

1. 署名: 您需要标出原文链接和作者信息;如果更改了原文章内容,需要说明.
2. 非商业使用: 您不得将本作品用于商业目的.
3. 相同方式共享: 基于本博客文章修改的作品需适用同一类型的许可协议.

**欢迎大家贡献关于R学习&数据处理方面学习笔记，感悟等!**

写作模板请见 `_draft` 这个文件夹。

## 文章参数配置说明

这个是文章的模板，应该放在`_posts`文件夹下，命名规则为`2016-09-30-title.md`，title需要全英文命名，请用`-`分隔，具体参考文件夹下已有的md命名。接下来详细说明一下上面的参数代表的意思以及正确的配置。

### author

这个参数传入作者的名字。这里的名字跟下面的`author_profile`是相关的，为了能正确的展示作者的名称和相关信息，你需要到`_data/authors.yml`添加相关的信息。需要注意的是:

- author 传入的名称应该与`authors.yml`里面的名称相一致，尤其注意空格什么的
- `authors.yml` 下面的wechat为微信二维码图片，应该放在`images/wechat`下面
- weibo与github给出的是域名简称，例如`http://weibo.com/brucezhaor`,`https://github.com/BruceZhaoR`最后面的

如果你的微博没有个性化域名，你需要去你的微博主页看看，然后复制一下后面的名称，例如：`u/1234567890`.

### date

这个参数是传入日期参数，其主要作用是文章的排序，以及在文章的末尾显示编辑日期，建议还是用英文的格式吧，毕竟逼格高一点，:blush:

### layout

文章的排版格式，一般博客文章就是使用`single`格式，其他页面有其他排版格式，这里就不具体介绍了，所有的排版在`_layouts`这个文件夹下，主要应用在`_pages`下面不同页面的排版，例如`archive,category,tags`等页面。

### header

这个是博客文章的大图 [header](/images/unsplash-gallery-image-1-th.jpg)
，放在文章的开头，这样逼格会瞬间高很多，当然如果你不想要的话可以直接删掉这个参数以及相关的设置。这里的参数很多，我需要详细的说明一下。

- `overlay_image`这个底图，也是必须的，默认的是主页图片。
- `overlay_color`如果底图的链接失效，就会展示这个颜色。
- `overlay_filter`如果找不到合适的图，可以玩颜色搭配,既可以是带透明色属性的`rgba(red,green,blue,alpha)`，alpha为透明度，范围0~1之间。也可以是`#123456`6位的hex颜色。
- `caption`为图片右下角的小字，标明图片来源，支持markdown语法的~
- `cta_label/url`为图片上的按键以及链接，样例为主页的`learn more`
- `padding`为图片的height，一般默认撑满第一个页面，只留下标题信息
- `excerpt`为底图上面，标题下面的副标题，支持markdown语法哦~~

这里有三种解决方案：第一种是纯图片格式的；第二种是图片加上透明色，营造一种朦胧的感觉；第三种就是玩色彩。这里我给出详细的例子供参考。

第一种，源码请见`_pages/archive.html`，预览见<http://bzrlc.tech/archive/>.只需要三个参数，`overlay_image, overlay_color &padding`，配置如下：

```
header:
  overlay_color: "#5c6266"
  overlay_image: unsplash-gallery-image-2.jpg
  padding: 10em
```
第二种，源码见`_pages/about.md`，预览见<http://bzrlc.tech/about/>.需要四个参数，在第一种的上面加上`overlay_filter`，其颜色为rgba，一定带透明色哦~ 配置如下：

```
header:
  overlay_color: "#000"
  overlay_filter: "#22b3eb, rgba(0,0,0,0.25)"
  overlay_image: mm-home-page-feature.jpg
  padding: 10em
excerpt: "关于博客的一些说明"
```
第三种，源码见`_pages/tag-archive.html`和`category-archive.html`，预览见<http://bzrlc.tech/tags/>和<http://bzrlc.tech/categories/>.参数与上面是一样的，只不过`overlay_filter`，其颜色为hex，还能调整颜色方向哦~~ 参考配置为：

```
header:
  overlay_color: "#000"
  overlay_filter: "to right bottom, #0091ff,#eae4e4,#1da794"
  overlay_image: mm-home-page-feature.jpg
  padding: 10em
```
需要说明的是，这里的`to right
bottom`意思是颜色从左上角向右下角辐射，相应的参数还有 `to left`,`to right`,`to
top`。自己可以用chrome的右键检查功能来进行颜色调试，不仅仅限于三种颜色哦~~ 

### comments

是否启用评论系统，默认为启用，其实你都可以不用加这个参数到yaml头文件中; 当然你如果不想要别人评论可以设置为false，个人认为没有评论系统完全没有活力啊，说不定你就碰到了传说中的神评论呢~

### categories

文章的分类，这里只能设置**一个**。你可以参考已有的文章的分类，如果没有，你可以自己新加一个。需要注意的是大小写的问题，默认首字母大写，如果是缩写就全部大写例如`ML`，建议使用`-`分隔或者大小写分隔，例如`Machine-Learning` / `MachineLearning` 都行。

### tags

文章所属标签。这里可以写多个，命名规则与categories一致。

### mathjax

是否启用latex公式。如果你的文章需要写一些latex公式，那么就需要设置为true，这里的默认为false，为了网页打开更快考虑的。需要说明的krmarkdown的latex语法公式有些不一样，一般用两个`$$`包围，`$$ 公式 $$`。如果写在段落里面，默认为嵌套公式；如果前后各空了一行，默认为居中专业版公式。另外，还有其他写法，如嵌套为 `\\( 嵌套公式 \\)` ，居中为 `\\[ 居中公式 \\]`。个人还是觉得两个dollar符号比较方便。

### highlight

是否引入RStudio的高亮方式，能够高亮出library之类的函数，其作用对象为：

```
{% highlight r %}

some R codes

{% endhighlight %}
```

之间的代码 some R codes。如果你是采用是：

```

    ```r
	some R codes 
	```
```

那就不需要引入highlight，需要设置`highlight: false`，后一种在github上面预览也是一样的，所以**推荐第二种写法**。

### description

文章的概括的描述，会呈现在主页文章列表的下面灰色字体部分，用于说明文章的主要内容，让人知道这篇文章大致是关于什么的。一般建议都稍微写一写 :blush:


### 最后

介绍终于详细地写完了，写了我好几个小时，反正在火车上无聊。。。最后祝大家国庆七天乐！


## 本地开发预览

` bundle exec jekyll serve --config _config.yml,_config.dev.yml`

## To Do List

- [x] 页面底端链接跳转 `<a target="_blank">`
- [x] 添加favicon.ico
- [x] 添加更多作者信息，如qq、WeChat、weibo等
- [x] post页面的简介，调整大小、颜色、倾斜，加上链接.参考链接：https://ropensci.org/blog/
- [x] tag, archive 页面重新设置(已加入anchor,用于跳转)
- [x] 增加about页面(放最后面) 介绍markdown的基本语法，加上emoji表情
- [x] collection 页面可以暂时去掉
- [x] home index 重新设计
- [ ] previou/next 增加标题说明？
- [ ] back to top 
- [ ] vim 换行是空格而不是tab？ !important
- [x] header image 的 linear-gradient重写 !important
- [ ] social icon 配色重新弄 


