---
title: Windows下Zookeeper伪集群搭建
date: 2016-05-13 15:33:02
tags: [Zookeeper, Apache]
category: 技术栈
---

# 概述
Zookeeper 分布式服务框架是 Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。

# 安装和配置详解

笔者使用的zookeeper版本是3.4.7，镜像下载地址http://apache.fayea.com/zookeeper/。
zookeeper的配置十分简单，下载之后解压缩。

ZOOKEEPER_HOME/conf/zoo.cnf
你可以直接复制zoo_sample.cfg，新建zoo.cfg文件，内容如下：

<!-- more -->

```
# The number of milliseconds of each tick 心跳间隔，单位毫秒
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take 初始同步阶段发送的心跳检测数
initLimit=10 
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement 发送和接收间隔的心跳数
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes. 快照数据位置
dataDir=D:\Tools\zookeeper-3.4.7\data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients 客户端连接的端口
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

```
启动Zookeeper
进入到bin目录，Shift+右键打开命令窗口，输入zkServer.cmd启动

# 集群模式
…

# 如何使用