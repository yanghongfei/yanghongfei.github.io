---
layout:     post
title:      "ZABBIX如何自定义模板记录"
subtitle:   " \"zabbix自定义模板介绍\""
date:       2019-03-20
author:     "Yangxiaofei"
header-img: "img/gitlab.jpg"
tags:
    - ZABBIX
---

## 										ZABBIX自定义模板



### Foreword

> 需求，需要对腾讯云机器进行数据库的监控。
>
> 解决，通过API获取相关数据库CPU/Mem/Disk数值，制作模板，利用Zabbix监控。



### 01. Import Template

> Template 可以随意从线上模板export出来一份即可

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1962i722wj30oh0g00tl.jpg)



### 02. Template Config

> 需要提前定义好Zabbix模板所需要的item,tigger等信息，本次主要为：CPU利用率，内存使用，磁盘使用。

#### 2.1 Item config 

`cpu_use_rate `

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1963fj293j30ut0n9jse.jpg)

`memory_use `

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1963rtzxaj30xr0nojs8.jpg)



`use_disk`

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1963xottcj30xx0o2jsi.jpg)

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1964587atj31he06ot9m.jpg)

#### 2.2 Tigger config

`cpu_tigger`

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1964fpiy1j30vn0fd3z5.jpg)



### 03. SaltMaster Config

> 登陆salt机器配置zabbix，通过指定脚本来获取值
>
> Zabbix统一安装目录：/usr/local/zabbix_agentd/
>
> Zabbix统一配置文件：/usr/local/zabbix_agent/etc/zabbix_agentd.conf.d/*.conf



#### 3.1 Zabbix Scripts 

> 示例，腾讯云CDB监控脚本

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 7/18/2018 4:57 PM
# @Author  : Fred Yang
# @File    : qcloud_cdb_monitor.py
# @Role    : 通过腾讯云监控获取CDB监控指标信息，参考官方文档：https://cloud.tencent.com/document/product/248/11006

import sys
import time
import datetime
import json
import random
import requests
from settings import TX_INFO
from qcloud_api import ApiOper

secret_key = TX_INFO['SecretKey']
secret_id = TX_INFO['SecretId']
# 对应的每一个接口都会一个单独的url地址
api_url = 'monitor.api.qcloud.com/v2/index.php?'


def get_dict(region,cdb_id,metrics):
    """
    填写字典参数，公共参数+接口参数
    :param region: str, 腾讯云区域
    :param cdb_id: str, 数据库的实例ID
    :param metrics: str,监控指标，如（CPU，Memory,Disk）
    示例：python3 qcloud_cdb_monitor.py ap-shanghai cdb-extlv472 cpu_use_rate
    :return:  int, CPU利用率等
    """

    end_time = datetime.datetime.now()
    start_time = end_time - datetime.timedelta(minutes=10)
    keydict = {
        #公共参数部分
        'Timestamp': str(int(time.time())),
        'Nonce': str(int(random.random() * 1000)),
        'Region': region,
        'SecretId': secret_id,
        # 'SignatureMethod': SignatureMethod,
        #接口参数部分
        'Action': 'GetMonitorData',
        'namespace': 'qce/cdb',
        'metricName': metrics,
        'dimensions.0.name': 'uInstanceId',
        'dimensions.0.value': cdb_id,
        'startTime': start_time.strftime("%Y-%m-%d %H:%M:%S"),
        'endTime': end_time.strftime("%Y-%m-%d %H:%M:%S")

    }
    return keydict


def get_response(result_url):
    """
    发送get请求
    :param result_url: 获取base64加密后的签名
    :return: 获取数据
    """
    res = requests.request('get', result_url)
    response = json.loads(res.text)
    #return response
    if len(response['dataPoints']) > 0:
        #print(response['dataPoints'])  # 返回得到的数据值
        print(max(response['dataPoints']))

    else:
        print("-0.00")

def index():
    if len(sys.argv) == 4:
        region = sys.argv[1]
        cdb_id = sys.argv[2]
        metrics = sys.argv[3]
        #填写字典内容，根据具体APi接口填写
        keydict = get_dict(region,cdb_id,metrics)
        #通过APi获得base64加密后的sign，获取请求的url
        result_url = ApiOper.run(keydict, api_url, secret_key)
        # 拿着base64加密后的签名请求资源
        get_response(result_url)

if __name__ == '__main__':
   index()

```



#### 3.2 Params Config

```shell
#将脚本相关的文件cp到zabbix指定目录
$ cp qcloud_api.py settings.py qcloud_cdb_monitor.py /usr/local/zabbix_agent/script/
$ cat /usr/local/zabbix_agent/etc/zabbix_agentd.conf.d/cdb-params.conf 
UserParameter=cdb.ins.base[*],/usr/local/bin/python3 /usr/local/zabbix_agent/script/qcloud_cdb_monitor.py $1 $2 $3
```

#### 3.3 Agentd  Restart 

```shell
$ /etc/init.d/zabbix_agentd restart
```



### 04.  ZabbixServer Add Host

> 准备工作完成后，开始添加云数据到zabbix

#### 4.1 Create Host

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1964ssqi3j310b0ofq49.jpg)



#### 4.2 Link Templates

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1964zm8qhj30p00adjs5.jpg)

#### 4.3 Macros Config

> 配置宏，这里需要传参到脚本 region,cdbid

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g19656ljqej30va0aw0to.jpg)



### 05 Monitor LastData

![](https://ws1.sinaimg.cn/large/005X1wn0gy1g1965cl2haj31h70lrdif.jpg)
