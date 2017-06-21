---
title: Linux awk命令详解
date: 2016-05-15 11:49:37
tags: [linux, awk]
category: linux命令详解
---

# 简介

awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

awk其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。实际上 AWK 的确拥有自己的语言：AWK 程序设计语言， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。

 # 使用方法

 ```
 awk '{pattern + action}' {filenames}

 ```
 其中 pattern 表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。
 通常，awk是以文件的一行为处理单位的。awk每接收文件的一行，然后执行相应的命令，来处理文本。

<!-- more -->

调用awk的命令行方式

 ```
 awk [-F  field-separator]  'commands'  input-file(s)

 ```
其中，commands 是真正awk命令，**[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。**

# awk入门实例

**示例1**：显示/etc/group所有group name


```
$ cat group // group内容
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
adm:x:4:root,adm,daemon
tty:x:5:
disk:x:6:root
lp:x:7:daemon,lp

```

```
$ awk -F ':' 'BEGIN {print "start.."} {print $1} END {print "end.."}' group
start..
root
bin
daemon
sys
adm
tty
disk
lp
end..

```
awk工作流程是这样的：先执行BEGIN操作，然后读取文件，读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符（-F 指定）划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。随后开始执行模式所对应的动作action,接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。
默认域分隔符是tab，-F指定分隔符为‘:’，所以$1表示group名称，$3表示group包含的用户,以此类推。

**示例2**：搜索/etc/group有root关键字的行并显示group name

```
$ awk -F ':' '/root/{print $1}' group
root
bin
daemon
sys
adm
disk
wheel

```

这是pattern的使用示例，匹配了pattern('/root/')的行才会执行action('print $1',没有指定action，默认输出每行的内容)。

*搜索支持正则，例如查找 `root`开头的行 `awk '/^root/' /etc/group`*。

# awk内置变量

awk有许多内置变量，这些变量可以被改变，下面给出了一些最常用的变量。

| 变量名 | 解释 | 
|---|---|
| ARGC | 命令行参数个数 | 
| ARGV | 命令行参数排列 | 
| ENVIRON | 支持队列中系统环境变量的使用 | 
| FILENAME | awk浏览的文件名 | 
| FNR | 浏览文件的记录数 | 
| FS | 设置输入域分隔符，等价于命令行 -F选项 | 
| NF | 浏览记录的域的个数 | 
| NR | 已读的记录数 | 
| OFS | 输出域分隔符 | 
| ORS | 输出记录分隔符 | 
| RS | 控制记录分隔符 | 

此外，$0变量是指整条记录，$1表示当前行的第一个域，$2表示当前行的第二个域，$NF表示当前行的最后一个域，以此类推。

**示例**：统计/etc/group/每行的文件名、行号、每行的列数、每行的内容：

```
$ awk -F ':' '{ print "filename: " FILENAME ", lines: " NR ", columns: " NF ", " $0 }' /etc/group
filename: /etc/group, lines: 1, columns: 4, root:x:0:root
filename: /etc/group, lines: 2, columns: 4, bin:x:1:root,bin,daemon
filename: /etc/group, lines: 3, columns: 4, daemon:x:2:root,bin,daemon
filename: /etc/group, lines: 4, columns: 4, sys:x:3:root,bin,adm
filename: /etc/group, lines: 5, columns: 4, adm:x:4:root,adm,daemon
filename: /etc/group, lines: 6, columns: 4, tty:x:5:
filename: /etc/group, lines: 7, columns: 4, disk:x:6:root
filename: /etc/group, lines: 8, columns: 4, lp:x:7:daemon,lp

```

# awk编程

## print和printf

awk中同时提供了print和printf两种打印输出的函数。

其中print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。

printf函数，其用法和c语言中printf基本相似,可以格式化字符串,输出复杂时，printf更加好用，代码更易懂。

上个示例使用`printf`替代`print`的写法：

```
$ awk -F ':' '{ printf("filename: %10s, lines: %s, columns: %s, %s\n",FILENAME,NR,NF,$0) }' /etc/group

```

## 指定分隔符

awk默认域分隔符是"tab"或者空格，使用-F指定分隔符，你可以同时指定多个分隔符，每个分隔符用'|'分割。

```
$ echo "a:b c,d" | awk -F " |,|:" '{print $1; print $2; print $NF}'
a
b
d

```

## BEGIN、END用法

awk工作流程是这样的：先执行BEGIN操作，然后读取文件，读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符（-F 指定）划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。随后开始执行模式所对应的动作action,接着开始读入第二条记录···直到所有的记录都读完，最后执行END操作。

**示例1**：有BEGIN有END：

```
$ awk -F ':' 'BEGIN {print "start.."} {print "group name: "$1; print "group users: "$3} END {print "end.."}' group
start..
group name: root
group users: root
group name: bin
group users: root,bin,daemon
group name: daemon
group users: root,bin,daemon
group name: sys
group users: root,bin,adm
group name: adm
group users: root,adm,daemon
....
end..

```

**示例2**：只有END：
```
$ awk -F ':' '{print "start.."} {print "group name: "$1; print "group users: "$3} END {print "end.."}' group
start..
group name: root
group users: root
start..
group name: bin
group users: root,bin,daemon
start..
group name: daemon
group users: root,bin,daemon
start..
group name: sys
group users: root,bin,adm
start..
group name: adm
group users: root,adm,daemon
....
end..

```

## 使用数组

### 一维数组

因为awk中一维数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)，值和关键字都存储在key/value格式的hash表中。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。

**示例**：我们要统计access_log日志每秒访问次数，并按倒序排：

```
$ cat access_log 
  1 12/May/2016:02:53:51    "GET /index" 
  2 12/May/2016:02:53:50    "GET /index" 
  3 12/May/2016:02:53:50    "GET /index" 
  4 12/May/2016:02:52:51    "GET /index" 
  5 12/May/2016:02:52:51    "GET /index" 
  6 12/May/2016:02:53:51    "GET /index" 
  7 12/May/2016:02:52:51    "GET /index" 
  8 12/May/2016:02:52:51    "GET /index" 
  9 12/May/2016:02:53:49    "GET /index" 
 10 12/May/2016:02:53:50    "GET /index" 
 11 12/May/2016:02:43:51    "GET /index" 
 12 12/May/2016:02:33:51    "GET /index" 
 13 12/May/2016:02:53:49    "GET /index" 
 14 12/May/2016:02:53:50    "GET /index" 
 15 12/May/2016:02:53:50    "GET /index" 
 16 12/May/2016:02:43:51    "GET /index" 
 17 12/May/2016:02:53:51    "GET /index" 
 18 12/May/2016:02:43:51    "GET /index"

```

```
$ awk '{ counts[$2]++; }; END{for(time in counts) print counts[time] " " time}' access_log | sort -n
1 12/May/2016:02:33:51
2 12/May/2016:02:53:49
3 12/May/2016:02:43:51
3 12/May/2016:02:53:51
4 12/May/2016:02:52:51
5 12/May/2016:02:53:50

```


### 多维数组

awk的多维数组的使用类似python中tuple数据结构，可以嵌套定义。

```
$ awk 'BEGIN{ for(i=1;i<=3;i++) {for(j=1;j<=3;j++) {arr[i,j]=i*j; print i,"*",j,"=",arr[i,j]}}}'
1 * 1 = 1
1 * 2 = 2
1 * 3 = 3
2 * 1 = 2
2 * 2 = 4
2 * 3 = 6
3 * 1 = 3
3 * 2 = 6
3 * 3 = 9

```

## 循环语句

awk的循环语句的写法和c语言中for、while、break、continue语法相同。

## awk内置函数

- **int**，把字符串转为整数
- **index**，查找字符串
- **length**，得到字符串、数组长度
- **match**，检测字符串1是否包含字符串2，包含则返回第一次出现的位置，否则返回0
- **rand**，生成随机数
- **split**，按照某个分隔符，对字符串进行切割，切割后的放到`thearray`中，返回分割后数组的长度
- **sub**，替换
> sub(a,b,s)，将s中出现的a字符串替换为b字符串

- **substr**，字符串截取
> substr(s, m, n)     s是要截取的字符串，m是开始点，从1开始，n是要截取的长度

- **toupper**，字符串转为大写
- **tolower**，字符串转为小写

