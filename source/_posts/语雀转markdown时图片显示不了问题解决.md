---
title: 语雀转markdown时图片显示不了问题解决
comments: true
categories:
  - 问题
tags:
  - markdown
  - blog
date: '2022-07-05 14:04:46 +0800'
abbrlink: 804121090
---

##  语雀转markdown时，图片显示不了  ##
### 原因是转换后的图片标签内多了这么一大串代码
![](http://rebp38war.bkt.clouddn.com/img/20220705135506.png)

![](http://rebp38war.bkt.clouddn.com/img/20220705135607.png)
### 按照网上分享的问题删掉这串代码，但是没有效果

### 后来在文档中加上这个标签就完事了
```html
<meta name="referrer" content="no-referrer" />
```