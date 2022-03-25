---
title: hexo新建文章分类
date: 2022-03-22 12:46:07
tags: hexo
toc: true
categories: hexo使用
---

如何创建分类
<!--more-->
## 1.创建分类选项
博客目录下执行
```
hexo new page categories
```
成功后提示
```
INFO  Created: E:\blog\junjieliao593.github.io\source\categories\index.md
```
可以看到生成的文件
```
---
title: categories
date: 2022-03-22 12:47:34
---
```
添加type: "categories"到内容中，添加后是这样的：
```
---
title: 文章分类
date: 2017-05-27 13:47:40
type: "categories"
---
```
至此，成功给文章添加分类，点击首页的“分类”可以看到该分类下的所有文章。
当然，只有添加了categories: xxx的文章才会被收录到首页的“分类”中。

## 2.给文章添加分类
添加tags和分类实例：
```
---
title: hexo新建文章分类
date: 2022-03-22 12:46:07
tags: hexo
toc: true
categories: hexo使用
---
```
