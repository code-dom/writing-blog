---

title: "Hugo使用"
subtitle: ""
date: 2025-08-17T13:04:40+08:00
draft: true

tags: []
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

license: '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'
---
- [ ] hugo基础使用
  - [ ] Quick start
    - [x] 创建本地hugo项目，如何写blog的环境
    - [x] 如何写第一个blog
    - [ ] 如何上传图片创建图床
      - [ ] https://www.yuhuizhen.com/2022/11/27/image-bed/
      - [ ] https://picgo.github.io/PicGo-Doc/zh/guide/config.html#github%E5%9B%BE%E5%BA%8A
      - [ ] https://zhuanlan.zhihu.com/p/340760172
- [ ] github pages托管hugo
  - [参考链接](http://hugo.opendocs.io/hosting-and-deployment/hosting-on-github/)
- [ ] 进阶
  - [ ] hugo主题设置、分类设置等
    - [ ] 如何组

# hugo基础使用

前提：安装hugo，[官网链接](https://hugo.opendocs.io/installation/)，自行根据自己电脑系统进行选择

### Quick start

1.创建项目

`hugo new site 目录名`

<img src="https://cdn.jsdelivr.net/gh/code-dom/picGo/img/202508181933604.png" alt="image-20250818193352473" style="zoom:50%;" />

2.创建具体blog

`hugo new xxx.md`

在文档中撰写blog内容

3.预览blog

`hugo server -D`

将展示所有blog，包括草稿，点击链接即可查看

