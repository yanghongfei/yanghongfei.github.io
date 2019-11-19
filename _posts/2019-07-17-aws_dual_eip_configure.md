---
layout: post
title: AWS - 记一次AWS EC2 多EIP、辅助IP配置过程
categories: [Linux, AWS]
description: Setup Dual Public IP for AWS EC2
keywords: Linux, AWS, EC2
---


## 										 记一次AWS EC2 多EIP、辅助IP配置过程


### 背景

> 一台海外的业务EIP不知是什么原因，部分端口被国家防火墙墙掉了，但是老的EIP则还需要使用，所以想到配置多个EIP，把遇到的坑做此记录。



 默认AWS EC2 启动完成后，是有一组PrivateIP 和 PublicIP的，AWS是支持一台EC2实例关联多个IP的（辅助IP）


#### AWS Console 分配私有IP并关联EIP


由于线上环境特殊、图标使用外部图片，涉权请联系作者删。

- 首先登陆AWS Console控制台
- 选择`网络`-`管理IP`-`分配新的IP`

![](/images/ec2-manage-ip-addresses-context.png)

![](/images/ec2-manage-ip-addresses-dialog.png)


- 分配新的EIP（不影响业务，请选择对应的辅助IP进行关联EIP，防止出错）

![](/images/ec2-eips.png)


#### 使机器能够识别你新增的IP

这里简单说明下，亲测，若是Amazon Linux, ec2-net-utils包默认可以处理掉所有事情,然后直接使用`sudo service network restart`进行直接识别到

```shell

ip addr 

sudo service network restart

ip addr

```


![](/images/amazon_linux_ipaddr.png)


但是，如果你是使用其余平台的CentOS系列机器，非Amazon Linux这样子操作是无法识别到的,这样就需要你手动进行配置


- 查看你的网络接口
```shell

ls -l /etc/sysconfig/network-scripts/*-eth?

```

- 拷贝一个新的，手动来操作
```
cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0:1

```
- 手动配置网卡信息

`vim /etc/sysconfig/network-scripts/ifcfg-eth0:1 `

```
BOOTPROTO=static
DEVICE=eth0
ONBOOT=yes
IPADDR=<private_ip>
NETMASK=255.255.255.0

```

- 启动

```shell

ifup eth0:1

```

**感谢**

原文链接：https://zzz.buzz/2018/04/07/setup-dual-public-ip-for-aws-ec2/  

官方文档：https://aws.amazon.com/cn/premiumsupport/knowledge-center/secondary-private-ip-address/
