---
title: hexo如何引用图片
date: 2021-08-13 11:31:16
subtitle:
categories: hexo
tags:
cover: /images/事务所.jpg
---

## 配置
安装`hexo-asset-image`后，在 `new post [title]`新建文章时，会在同一目录下生成同名文件夹
```
npm install hexo-asset-image --save
```
![同名图片目录](20210813150044.png)

将资源放至*测试图片*目录下，在*测试图片.md*中直接引用图片即可，无需相对路径或绝对路径

### 标签语法
`{% asset_img 灵笼白月魁.jpg '月魁yyds' post-image 100 100 %}`

``` html
<img src="/灵笼白月魁.jpg" class="content-image" width="500" height="100" title="月魁yyds" />
```
</br>

{% asset_img content-image 灵笼白月魁.jpg 500 100 月魁yyds  %}

ex:`hexo的标签语法vscode的 markdown预览不支持`

---------

### markdown语法
```
![白月魁](灵笼白月魁.jpg)
```
![白月魁](灵笼白月魁.jpg)


## 注意：cover的配置
显示在首页的img需要将资源放至`/source/images/`下，通过如下方式引用
```
---
title: hexo如何引用图片
date: 2021-08-13 11:31:16
cover: /images/事务所.jpg
---
```