---
layout: post
title: "Jekyll搭建GitHub静态博客"
date: 2020-10-16
author: 'dax'
categories: "note"
typora-root-url: ../../bestdax.github.io
---



## 为什么需要一个自己的空间

现在网上有很多可以写博客的地方，提供的功能也很丰富，为什么我们还要折腾自己弄一个GitHub上的空间呢？我主要原因有两个：

1. 自己弄这件事情够折腾😄️

2. 别人的地盘上有别人的规矩，老子不吃这一套。

   其实还可以加上第三点，自己弄，定制程度高，虽然会多花些时间和精力，但是可以做出真正自己喜欢的东西。配合`Git`可以对博客进行一次一次的修改，直到写出满意的内容为止。

   

   

## 准备工作

首先，第一步需要有一个GitHub账号。这个只要注册一下就可以。

然后，根据[GitHub Pages](https://pages.github.com/)上指示的步骤就可以写个`Hello World`的网页啦。

当然，不可能以后的博客都写成`html`格式的来提交，这就轮到`Jekyll`出场啦。

## Jekyll的简单设置

<img src="/assets/images/image-20201018150301878.png" alt="image-20201018150301878" style="zoom:50%;" />

这个截图是`Jekyll`官网上的介绍，安装，建站，和本地预览，相当简单。

就是建站完了以后，需要对`_config.yml`和`gemfile`文件进行一些简单的设置。

`gemfile`的修改：

```
gem "jekyll", "~> 4.1.1" # 把这一行去掉
gem "github-pages", group: :jekyll_plugins # 把这一行的注释标志去掉
```

`_config.yml`里面把网站名字，描述等一些相关信息填上。另外，要设置一个文件夹来放置图片。

```
defaults:
  - scope:
      path: "assets/images"
    values:
      image: true
```

需要在网站的文件夹里新建文件夹assets/images，以后添加的图片就放这个文件夹里。

这样就建好一个站了，可以试试用`bundle exec jekyll serve`命令然后在浏览器里用上面图片中的那个地址来预览网站了。

`Jekyll`有个初始的welcome页面提供的，需要注意的是自己新建的post要有格式抬头如下：

```
---
layout: post
title:  "Welcome to Jekyll!"
date:   2020-10-16 14:26:20 +0800
categories: jekyll update
---
```

并且，**博客日志的文件名是有固定格式的**。`YEAR-MONTH-DAY-title.MARKUP`，其中，年份是4位数，月日都是两位数，标记的格式一般用`markdown`的话写`.md`就可以了。

## 更换Theme

虽然现成的主题已经是多次优化的结果，但是具体到个人，还是或多或少需要一些个性化的调整的。修改主题需要以下操作

1. 添加下面内容到 `Gemfile`，把主题的名字替换成想要的:

```
gem "bulma-clean-theme"
```

2. 添加下面内容到 `_config.yml`:

```
theme: bulma-clean-theme
```

3. 然后执行:

```
$ bundle
```

以上是把主题下载到本地的方法。要在GitHub上面使用的话，只要在`_config.yml`里加上这一句就好了。

```
remote_theme: chrisrhymes/bulma-clean-theme
```

## 定制Theme

默认的`minima`主题看起来还是有点太寡淡了，心有不甘的继续折腾。之前没有成功是因为`minima`主题有`post`、`home`以及`page`三个`layout`，而一般的主题只有一个`post`外加一个`default`，所以`build`的时候会出错。我于是想到能不能把`minima`主题`clone`下来，然后进行一些微调。说干就干，`clone`完后发现有几个是需要拷贝到网站的文件夹里的。

<i>对于Git还不怎么会，不知道在同一个文件夹里用Git来克隆会不会有干扰，就选择复制了。</i>



<img src="/assets/images/image-20201019133124640.png" alt="image-20201019133124640" style="zoom:50%;" />

其中的_sass文件夹从名字就可以看出是控制样式的。然后我又在`assets`里面找到了一个`css`文件夹。

打开`style.scss`文件可以看到以下内容：

```
---
# Only the main Sass file needs front matter (the dashes are enough)
---

@import
  "minima/skins/{{ site.minima.skin | default: 'classic' }}",
  "minima/initialize";
```

根据这个内容我找到`_sass`文件夹里面的`skin`文件夹，里面有这四个皮肤文件。

<img src="/assets/images/image-20201019134116119.png" alt="image-20201019134116119" style="zoom:50%;" />

试着将上面的`default: 'classic'`改为`solarized`然后在本地的服务器上试了一下，哦耶，成功了!

默认的格式没段首缩进，大标题也不居中，从中文的角度看起来比较别扭。就琢磨着如何变更一下设置。在_sass文件夹里面有一个`custom-sytle.sass`，里面的说明是定制格式的话可以放在这个文件里，试着在这个文件里面写了如下内容：

```html
p {
	text-indent: 2em;
}

h1 {
	text-align: center;
}
```

这下子看起来舒服多了。

## 配合Typora

用`Typora`配合书写`markdown`是一件很惬意的事情，最重要的是插入图片会非常方便。首先要把图片的文件夹给设置一下：

<img src="/assets/images/image-20201103153039481.png" alt="image-20201103153039481" style="zoom:50%;" />

然后还要设置图片的根目录，在`格式`→`图片`→`设置根目录`中把根目录设置为网站的根目录文件夹，这样以后插入的图片就能在GitHub上正确显示了。

