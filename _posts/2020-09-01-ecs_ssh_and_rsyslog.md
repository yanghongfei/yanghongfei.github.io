---
layout: post
title: ECS - 记一次因rsyslog配置导致的SSH卡顿现象
categories: [Linux, Aliyun]
description: SSH stall caused by rsyslog
keywords: Linux, ALiyun, ECS
---


## 										 记一次因rsyslog配置导致的SSH卡顿排查过程


### 背景

> 因为机器运行网络问题需要进行迁移，将机器从经典网络迁移到VPC网络中


- 制作镜像
- 购买实例
- 开启实例
- 修改配置
- 启动服务

但是迁移完成后，发现机器连接ssh很卡，甚至无法进行正常敲命令。



#### 处理过程记录

> 下面记录是开始了折腾，由于之前有踩过ssh卡的坑,所以就针对这几个方向无脑去做了，先试下

```
$ service rsyslog status
$ service nslcd status
$ service rsyslog stop
$ chkconfig rsyslog off
$ service nslcd stop
$ chkconfig nslcd off
$ vim /etc/nsswitch.conf 
sudoers: files ldap   #这是因为有次也是因为卡 ldap验证导致的
```

停止了以上服务后，发现是恢复了正常，开始找原因发现 是因为rsyslog配置了老的IP，导致网络不通 一直上报卡在那里。

```
vim /etc/rsyslog.conf

#有个*.* 配置到一个其他网络的内网地址(不通)


#删除掉，开启rsyslog
/etc/init.d/rsyslog start 
chkconfig rsyslog on
```
