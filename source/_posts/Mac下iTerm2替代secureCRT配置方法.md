---
title: mac下iterm2替代secureCRT配置方法
date: 2016-10-29 17:24:00
tags: [Mac, iTerm2]
category: Mac效率提升
---

最近在折腾iterm2的自动登录问题，由于之前一直在windows使用xshell作为我的ssh客户端，习惯了自动登录以及在远程机器上执行本地脚本（script），刚接触mac上iterm2很崩溃，虽然iterm2的分屏等操作极大方便了程序员的开发，但是日常经常通过跳板机登录线上机器，使我对于输入机器名密码登录很是头疼。xshell可以编辑快捷按钮并且自定义脚本方式达成目的，iterm2可是需要折腾一番。

# Session复制
windows下xshell和secureCRT等ssh客户端有个很大的优势就是可以复制会话，可以不用重复输入机器密码。

ssh本身也提供了一种快捷的方式来解决这个问题，在 `~/.ssh/config` 文件中加入

```
Host *
ControlMaster auto
ControlPath ~/.ssh/master-%r@%h:%p
```
这样下次登录同一个站点时，即可复用之前的会话信息，我们可以在`~/.ssh/`目录看到
![](/uploads/1942b8af4d43f54246c29abba.png)
这代表每次登录这个站点的时候其实是复用之前的会话句柄。

<!-- more -->

# 登录远程机器

登录远程机器通常有两种，一种是通过账号密码登录，另一种是基于密钥的安全认证，`ssh密钥认证自动登录`正是通过这种方式的安全认证，但是配置起来比较麻烦，这里不推荐，感兴趣的同学可以看这里[配置ssh密钥认证自动登录](https://segmentfault.com/a/1190000000481249)。下面我们介绍通过密码登录的配置方式。

## 可以直连的机器

xshell和secureCRT另一个很方便的功能就是，我们可以将登录机器的信息保存并使用快捷操作（保存成脚本或快捷命令），这样每次登录机器时就不用敲一堆机器名密码，最头疼的是我们根本也记不住这些用户名密码，iterm2有没有相应的解决方案呢？答案是肯定的。

在 `~/.ssh/config` 文件中加入

```
Host <alias>
    Hostname <ip>/<hostname>
    Port <port>
    User <username>
```
这样当我们输入`ssh alias`的时候，其实执行的是`ssh username@ip/hostname:port`，这样就很方便得登录到远程机器。

## 无法直连的机器

大公司经常通过跳板机登录到目标机器上，为了安全考虑，我们是无法直接连接线上机器的，通过上面的方式无法解决我们的问题，那有没有其他方式呢？答案是有的，我们可以把这个问题分成两种情况：

### 静态密码登录

有些公司的跳板机是静态密码，也就是使用固定密码登录，可以分为以下几个步骤：

1.写一个简单的expect脚本，如`remote.exp`:

```
#!/usr/bin/expect  

set timeout 30 

# 跳板机
set jumphost relay01.baidu.com
set jumppasswd root
set jumpuser root
set jumpport 22

# 登录跳板机
spawn ssh $jumpuser@$jumphost:$jumpport

# expect输出信息请自定义，如不需要密码，下段可省略
expect {  
    "(yes/no)?"  
    {send "yes\r"}  
    "password:"  
    {send "$jumppasswd\r"}  
}

# 目标机器信息
set host dev.baidu.com
set passwd xxx
set user xxx
set port 22

# 登录目标机器
send "ssh $user@$host:$port"
# expect输出信息请自定义，如不需要密码，下段可省略
expect {  
    "(yes/no)?"  
    {send "yes\r"}  
    "password:"  
    {send "passwd\r"}  
}
# 结束控制
interact 

```
提示： 
- 命令字符串结尾别忘记加上“\r”或“\n”，相当于回车
- expect后根据自己机器的实际输出，定义判断字段

2.将脚本复制到`~/.ssh`目录下
```
mv remote.exp ~/.ssh
```

3.设置调用命令

iTerm -- preferences 打开设置界面，配置如下：
![](/uploads/5f7b238a183ff66cf3b57342c.png)

### 动态密码登录

大公司中，使用动态密码登录跳板机的情况比较常见，动态密码一般是自己设置的6位密码+token的动态6位数字，这种方式看似复杂，其实我们可以通过两种情况看:
1. 已经通过动态密码登录过跳板机：【复制会话】会实现自动登录，配置方式同上；
2. 有的同学比较任性，之前没有登陆过跳板机，能不能实现一次操作直接登录跳板机和目标机器呢？可以，但需要高阶一点的`expect`语法，写法如下：
```
#!/usr/bin/expect

set timeout 30

# 跳板机
set jumphost relay01.baidu.com
set jumpuser root
set jumpport 22

# 登录跳板机
spawn ssh $jumpuser@$jumphost:$jumpport

# 目标机器信息
set host dev.baidu.com
set passwd xxx
set user xxx
set port 22

interact {
    "off" {
        send "ssh $user@$host:$port\n"
        # exec ssh $user@$host:$port
        interact
    }
}
```
提示：`off`为用户设置的远端机器的别名，登录跳板机后，输入“off”登录目标机器。


这种方式可能还不够优雅，下面介绍**终极解决方案**：

```
#!/usr/bin/expect

# 跳板机
set jumphost relay01
set jumpuser xiashanshan
set jumpport 22

# grab the password
stty -echo
send_user -- "Password for $jumpuser@$jumphost: "
expect_user -re "(.*)\n"
send_user "\n"
stty echo
# set pass
set pass $expect_out(1,string)

#exec ssh $jumpuser@$jumphost

# 登录跳板机
spawn ssh $jumpuser@$jumphost

expect {
    "*password:*" { send "$pass\n" }
    "*ssl*" { send "\n" }
    default { exit 1 }
}

# 目标机器信息
set host idev.baidu.com
set user root
set password root

send "ssh --matrix $user@$host\n"

expect {
    "*password:*" { send "$pass\n" }
    default { exit 1 }
}
interact

```
提示：如果已经登陆过跳板机，控制台输出`Password for $jumpuser@$jumphost: `提示输入密码时，可以直接回车跳过。


# rz、sz文件上传下载

首先需要安装`lrzsz`，如果没有安装`brew`，参考这里:[http://brew.sh/index_zh-cn.html](http://brew.sh/index_zh-cn.html)

```
brew install lrzsz
```
然后下载iterm2-zmodem：
```
cd /usr/local/bin
wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-send-zmodem.sh
wget https://raw.github.com/mmastrac/iterm2-zmodem/master/iterm2-recv-zmodem.sh
```
打开`preferences → profiles`，选择某个profile，之后继续选择advanced → triggers，添加编辑添加如下triggers：

| Regular Expression | Action | Parameters |
| ------| ------ | ------ |
|rz waiting to receive.\*\*B0100|Run Silent Coprocess|/usr/local/bin/iterm2-send-zmodem.sh|
|\*\*B00000000000000|Run Silent Coprocess|/usr/local/bin/iterm2-recv-zmodem.sh|

`zmodem`原作者：[https://github.com/mmastrac/iterm2-zmodem](https://github.com/mmastrac/iterm2-zmodem)

参考文章：
1.[https://linux.die.net/man/1/expect](https://linux.die.net/man/1/expect)
2.[https://www.pantz.org/software/expect/expect_examples_and_tips.html](https://www.pantz.org/software/expect/expect_examples_and_tips.html)

