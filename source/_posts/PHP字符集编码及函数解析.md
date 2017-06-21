---
title: PHP字符集编码及函数解析
date: 2016-05-13 15:34:37
tags: [php]
category: 技术总结
---

# php字符长度函数


> utf-8编码下中文3个字节，英文1个字节
> gbk编码下中英文统一2个字节

## strlen
返回给定的字符串 string 的字节长度。

例如：

```
$str = '新人求脸熟';
echo strlen($str)."\n";

print 15;

```

引用 [strlen]()

<!-- more -->

## mb_strlen

mixed mb_strlen ( string $str [, string $encoding = mb_internal_encoding() ] )
返回具有 encoding 编码的字符串 str 包含的字符数。 多字节的字符被计为 1。

文件编码：utf-8情况下
例1：

```
$str = '新人求脸熟';
echo mb_strlen($str)."\n";

print 5;

```
例2 ：

```
$str = '新人求脸熟';
echo mb_strlen($str, 'gbk')."\n";

print 8;

```
这是由于utf-8编码下，str包含5个汉字，占用15个字节，如果用gbk编码，则会返回15/2 = 8个字符长度。

引用 [mb_strlen]()

# php字符截取函数

待续…

# 参考资料
[阮一峰字符编码笔记]()