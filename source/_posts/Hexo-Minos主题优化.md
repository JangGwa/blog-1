---
title: Hexo Minos主题优化
date: 2017-05-05 13:57:15
tags: [Hexo, Minos]
category: Hexo
---

### Tags/Categories页面无法显示问题

- 解决方法：

在生成的`source/tags/index.md`中加入
```
layout: tag // 引用theme下的layout中tag.ejs
```
以及`source/categories/index.md`中加入
```
layout: categories // 引用theme下的layout中category.ejs
```