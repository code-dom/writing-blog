---
title: "Hugo使用"
subtitle: ""
date: 2025-08-17T13:04:40+08:00
draft: false

tags: []
categories: [工具]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

license: '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'
---
## 前置项

前提：安装hugo，[官网链接](https://hugo.opendocs.io/installation/)，自行根据自己电脑系统进行选择

## Quick start

1.创建项目

`hugo new site 目录名`

<img src="https://cdn.jsdelivr.net/gh/code-dom/picGo/img/202508181933604.png" alt="image-20250818193352473" style="zoom:50%;" />

2.创建具体blog

`hugo new xxx.md`

在文档中撰写blog内容

3.预览blog

`hugo server -D`

将展示所有blog，包括草稿，点击链接即可查看。

示例如下：

<img src="https://cdn.jsdelivr.net/gh/code-dom/picGo/img/202508181938777.png" alt="image-20250818193843578" style="zoom:50%;" />

## 安装主题

接下来安装主题,我们直接使用Hugo推荐的一些主题。比如说我使用的是blackburn这个主题:

```
git clone https://github.com/olOwOlo/hugo-theme-even.git themes/even
```

将主题git clone到themes/even目录下,在config.toml中配置:

```
theme = "even"
```

这样主题就安装好了。

## 如何使用github托管blog

按照hugo官方文档配置workflow：[参考链接](http://hugo.opendocs.io/hosting-and-deployment/hosting-on-github/)

配置完成后每次git push自动触发更新。

## 如何创建图床

我选择使用picGo来帮助管理图片上传，具体参考[picGo配置图床，官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A)

配置完成后可以在picGo的app上使用，例如：

![image-20250819091712639](https://cdn.jsdelivr.net/gh/code-dom/picGo/img/202508190917739.png)

如果使用的是typora写作可以参考，配置图片上传功能[PicGo+Typora搭建个人笔记系统](https://zhuanlan.zhihu.com/p/340760172)

## 参考资料

http://shuzang.top/2019/hugo-blog-article-write/

https://yuanyi-au.github.io/posts/hugo/

https://gdzhu8023.github.io/post/buildblog/

https://www.yuhuizhen.com/2022/11/27/image-bed/

https://picgo.github.io/PicGo-Doc/zh/guide/config.html

https://zhuanlan.zhihu.com/p/340760172

**切换主题**可参考：[使用hugo搭建博客](https://jellyzhang.github.io/%E4%BD%BF%E7%94%A8hugo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)
