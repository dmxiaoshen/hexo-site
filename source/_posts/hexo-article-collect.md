---
title: Hexo博客在icarus主题下写作的常用功能
date: 2015/04/27
toc: true
---

博客由hexo搭建，使用icarus主题，该主题有丰富的特性，稍作记录，以备后忘。
<!-- more-->

## 分类

```yml
---
categories:
- 笔记
- Hexo
---
效果就是[Hexo]是[笔记]的二级目录分类

同理，多标签则可以写作
---
tags:
- 标签1
- 标签2
---
```

## 主页文章banner

```yml
---
banner: 图片url
---
```

## 文章缩略图

```yml
---
thumbnail: 缩略图url
---
```

## 文章目录

```yml
---
toc: true
---
```

## 主页文章多图

```yml
---
photos:
- 图片地址1
- 图片地址2
- 图片地址3
---
```
