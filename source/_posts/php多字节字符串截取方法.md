---
title: php多字节字符串截取方法
date: 2016-11-07 15:05:35
tags: [php]
category: 技术总结
---

PHP的substr()方法多用于截取英文等单字节文件，对于中文字符串通常会返回各种乱码，多字节字符串（中文）在不同编码下的占用字节不同，针对这种情况，下面给出一个参考实现，需要的同学根据自己的需求可以在此基础上修改：

```php
/**
 * 多字节字符串截断方法
 * @param $str 原始字符串
 * @param $start 起始位置，多字节字符长度视为1
 * @param $len 长度，多字节字符长度视为1
 * @param string $charset 原始字符串编码
 * @return string 处理后字符串
 */
function mb_substr($str, $start, $len, $charset = 'utf8') {

    if ($charset == 'utf8' || $charset == 'utf-8') {
	    $mb_step = 3;
    } else {
	    $mb_step = 2;
    }

    // 字符串字节长度
    $str_len = strlen($str);

    // 字节偏移量
    $offset = 0;
    for ($i = 0; $i < $start; $i++)
    {
	    if ($offset >= $str_len) break;
	    $char = ord($str[$offset]);
	    if($char <= 0x7F) {
		    $offset++;
	    } else {
		    $offset += $mb_step;
	    }
    }
    // 字节长度
    $mb_len = 0;
    for ($i = 0; $i < $len; $i++)
    {
	    if ($offset+$mb_len >= $str_len) break;

	    $char = ord($str[$offset+$mb_len]);
	    if($char <= 0x7F) {
		    $mb_len++;
	    } else {
		    $mb_len += $mb_step;
	    }
    }
    // 截取字符串
    return substr($str, $offset, $mb_len);
}

```