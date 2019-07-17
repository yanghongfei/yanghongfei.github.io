---
layout: post
title: 开发 - 记一次Tornado异步HttpClient客户端使用【附源码】
categories: Tornado
description: 异步非阻塞HTTPClient客户端踩坑记录
keywords: Tornado
---


## 										记一次Tornado异步HttpClient客户端的使用


### 前言

> 由于项目都是Tornado写的，其余`libs`里面都是直接用`request`请求，本文主要记一次`tornado`异步客户端使用

[tornado.httpclient](https://tornado-zh.readthedocs.io/zh/latest/_modules/tornado/httpclient.html#HTTPClient)  

[异步HTTP客户端中文文档](https://tornado-zh.readthedocs.io/zh/latest/httpclient.html)


这里废话不多说，直接代码示例，方面后续再用到且能很快的入手


#### request模式下GET请求

简单记录下`request`模式下怎么实现带`auth_key`的`GET`请求

```python

import requests
import json

#认证auth_key
auth_key = 'xxxxx' 
#接口
accept_task_url = 'https://codo.domain/api/task/v2/task/accept/'
#GET请求
req1 = requests.get(accept_task_url, cookies=dict(auth_key=auth_key))
#拿返回
csrf_key = json.loads(req1.text)['csrf_key']

```

#### Tornado HttpClient模式下GET请求

再来记录下`tornado`的异步http客户端`http_client`是怎么实现带`auth_key`的`GET`请求

```python

from tornado import httpclient


class HttpClientTestHandler(tornado.web.RequestHandler):

    async def get(self, *args, **kwargs):

        #认证auth_key
        auth_key = 'xxxxx' 

        #接口
        task_url = 'https://codo.domain/api/task/v2/task/accept/'

        #异步HTTP客户端
        http_client = httpclient.AsyncHTTPClient()

        #设置cookie, web可以将整个headers打出来看一下里面的值：print(self.request.headers)
        cookie = {"Cookie": 'auth_key={}'.format(auth_key)}
        
        #请求
        csrf_response = await http_client.fetch(task_url, method="GET", raise_error=False, headers=cookie)

        #response.error是错误信息，我直接判断的状态码
        if response.code != 200:
        return self.write(dict(code=-3, msg="错误码:{}".format(response.code)))

        #返回的数据都在body里面
        response_data = json.loads(response.body.decode('utf-8'))
        if response_data.get('code') != 0:
            return self.write(dict(code=-3, msg="权限错误:{}".format(response_data.get('msg'))))
        

```


#### request模式下POST请求

简单记录下`request`模式下带`auth_key`和`csrf_key`的`POST`请求

```python


import requests
import datetime
import sys
import json

def post_task():

    #认证
    auth_key = 'xxxxx'

    #body的数据
    the_body = json.dumps({'exec_time': '2019-07-17 19:04:32', 'temp_id': 24, 'schedule': 'new', 'submitter': '杨红飞', 'args': "{'TAGGER_NAME': 'Zabbix agent on Zabbix server is unreachable for 5 minutes', 'HOSTNAME': 'Zabbix server', 'HOSTIP': '127.0.0.1'}", 'hosts': "{1: '127.0.0.1', 2: '127.0.0.1'}"})

    #接口
    accept_task_url = 'https://codo-v1.domain.com/api/task/v2/task/accept/'

    #request get先拿到csrf_key
    req1 = requests.get(accept_task_url, cookies=dict(auth_key=auth_key))
    csrf_key = json.loads(req1.text)['csrf_key']
    
    #设置cookie  auth_key + csrf
    cookies = dict(auth_key=auth_key, csrf_key=csrf_key)

    #request post请求
    req = requests.post(accept_task_url, data=the_body, cookies=cookies)
    
    #请求结果
    print(json.loads(req.text))


if __name__ == '__main__':
    post_task()


```

#### Tornado HttpClient模式下POST请求

再来记录下`tornado`的异步http客户端`http_client`是怎么实现带`auth_key` + `csrf_key` 的`POST`请求

```python

class HttpClientTestHandler(tornado.web.RequestHandler):

    async def post(self, *args, **kwargs):

        #认证auth_key
        auth_key = 'xxxxx' 
        
        #接口
        task_url = 'https://codo.domain/api/task/v2/task/accept/'

        #body的数据
        the_body = json.dumps({'exec_time': '2019-07-17 19:04:32', 'temp_id': 24, 'schedule': 'new', 'submitter': '杨红飞', 'args': "{'TAGGER_NAME': 'Zabbix agent on Zabbix server is unreachable for 5 minutes', 'HOSTNAME': 'Zabbix server', 'HOSTIP': '127.0.0.1'}", 'hosts': "{1: '127.0.0.1', 2: '127.0.0.1'}"})

        #异步HTTPClient
        http_client = httpclient.AsyncHTTPClient()

        #设置cookie  auth_key
        cookie = {"Cookie": 'auth_key={}'.format(auth_key)}

        #GET请求先拿到csrf_key
        csrf_response = await http_client.fetch(task_url, method="GET", raise_error=False, headers=cookie)
        csrf_key = csrf_response_data.get('csrf_key')

        #设置cookie  auth_key + csrf_key, 这里你可以print(self.request.headers)看下他的格式
        #auth_key csrf_key等信息都是在headers里面的，分号隔开的，我们也要按照这个格式来
        cookie = {"Cookie": 'auth_key={}; csrf_key={}'.format(auth_key, csrf_key)}

        #异步http_client请求
        response = await http_client.fetch(task_url, method="POST", body=the_body,raise_error=False, headers=cookie)

        #返回的数据都在body里面
        response_data = json.loads(response.body.decode('utf-8'))

```



#### 再来记录一些最常用到的返回值

**request**
- res.url： 获取请求url
- res.status_code: 状态码
- res.text：body数据
- json.loads(res.text): JSON格式

**tornado http_client**

- response.code： 状态码
- response.error: 错误信息
- response.body: Body数据
- json.loads(response.body.decode('utf-8'))：JSON Body数据

