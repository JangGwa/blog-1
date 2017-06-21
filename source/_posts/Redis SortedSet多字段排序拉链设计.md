---
title: Redis SortedSet多字段排序拉链设计
date: 2017-03-07 15:05:35
tags: [Redis, php]
category: 技术总结
---


## 应用场景描述

> pb热门回复列表，规则为按照三个字段综合排序(优先按照点赞数排序、其次按照楼中楼数排序、最后按照最后回复时间排序)：
> - 回复点赞数`agree_num`
> - 回复楼中楼数`comment_num`
> - 回复的楼中楼最后回复时间`last_reply_time`

## 实现思路

这里探讨的主要有两种实现方案：

- 方案一：使用`DB/DDBS`的`order by` 多级字段排序；
- 方案二：`redis`的SortedSet提供类排行榜实现，底层使用skip list，排序和查询时间复杂度较低；

<!-- more -->

优缺点分析：

| 方案 |优点 | 缺点 |
| --- | --- | --- |
| 方案一 | 代码实现简单 | 1、数据量膨胀后性能迅速降低<br>2、 不支持高并发场景<br>3、写qps较高时重建索引对性能损耗较大 |
| 方案二 | 1、查询时间复杂度为O(log(n))，耗时稳定 <br>2、内存型存储，支持较高qps | 占用redis集群内存资源 |


## SortedSet

Sorted set底层基于跳跃表实现，查询的时间复杂度在O(log(n))。

Redis中的SortedSet根据score的64位双精度浮点数的参数实现排序，可以表示浮点数或整数类型，作为浮点数score的取值范围为54位， 但是在实际应用中推荐将score当做64位长整型来使用。

原因很简单: long的取值范围要大于double。
> long范围为[-(2^53), +(2^53)]，double范围为[-(2^63), +(2^63)-1]


## 实现方案

### 设计探讨
由于需要使用三个字段（`agree_num/comment_num/last_reply_time`）作为排序依据，所以如何把3个字段合并为1个字段计入score，是解决问题的关键。

当score作为`64bit`长整型使用时，我们假设前提约束条件（楼层点赞数小于20万、楼中楼数小于20万）：
> `agree_num` <= 2097151 (2^21-1)
> `comment_num` <= 4194303(2^22-1)

这样`agree_num`和`comment_num`总共需要占用*44bit*，`last_reply_time`字段只有*20bit* 可用。

但根据unix时间戳（精确到s），10位数字时间戳最大值至少需要*33bit*存储，关键问题转为如何优化时间戳的存储。

### 存储方案

和业务pm沟通之后，产品上可以接受按回复时间排序时只精确到天级别，那么我们可以根据unix时间戳得到日期，如`20170329`，更极致的做法是将年份前两位`20`也去掉，只保留`170329`，10进制6位数字恰好使用*20bit*可以满足存储需求。

### 实现逻辑

由于热门回复的排序逻辑中三个字段有不同的优先级，我们最先想到的实现方案是按bit位分割，优先级高的字段占据较高的bit段，优先级低的字段在较低的bit段。

在64位机器下，php integer类型取值范围为*[-(2^63, +2^63-1]*，而热门回复的三个排序字段占位已优化为：
> `agree_num` 22bit
> `comment_num` 22bit
> `last_reply_time` 20bit

所以设置Sorted set的score实现代码就很简单了：
``` php
$score = (int)$agree_num << 42 | (int)$comment_num << 20 | (int)$dayOfDate;
$arrReq = array(
    'key' => self::REDIS_KEY_PREFIX. $arrInput['thread_id'],
    'members' => array(
        array(
            'score' => $redisScore,
            'member' => $arrInput['post_id'],
        ),
    ),
);
```

实现思路：`agree_num`左移*42bit*，占位最高*22bit*，`comment_num`左移*20bit*，占位中间*22bit*，`dayOfDate`占位最低*20bit*。


ps: 如何根据score获取原字段值？
```php
$agree_num = $score >> 42;
$comment_num = $score >> 20 & ~(PHP_INT_MAX << 20);
$dayOfDate= $score & ~(PHP_INT_MAX << 20);
```
> 完美~

## 性能测试
暂无


## 参考文档

- https://redis.io/topics/indexes
- http://php.net/manual/zh/language.types.integer.php