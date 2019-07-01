---
layout: wiki
title: Python
categories: Python
description: Python中的一些记录。
keywords: Python
---

### 1. Python基本使用
#### 1.1 安装Python3

```
python3.0安装

python链接

在 CentOS 7 中安装 Python 依赖

$ yum -y groupinstall development 
$ yum -y install zlib-devel 
$ yum install -y python3-devel openssl-devel libxslt-devel libxml2-devel libcurl-devel 
在 Debian 中，我们需要安装 gcc、make 和 zlib 压缩/解压缩库

$ aptitude -y install gcc make zlib1g-dev
运行下面的命令来安装 Python 3.6：

$ wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz
$ xz -d  Python-3.6.3.tar.xz
$ tar xvf Python-3.6.3.tar
$ cd Python-3.6.3/
$ ./configure
$ make && make install

# 查看安装
$ python3 -V
pip3安装
```



```
yum groupinstall "Development Toos" -y
yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel -y
wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz
tar -xzvf Python-3.6.0.tgz -C  /tmp
cd  /tmp/Python-3.6.0/
./configure --prefix=/usr/local/python
echo PATH='/usr/local/python/bin/:$PATH' >> /etc/profile
source /etc/profile
```


```
yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel -y
yum -y install gcc make python-devel libffi-devel
wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz
tar -xzvf Python-3.6.0.tgz -C  /tmp
cd  /tmp/Python-3.6.0/
./configure && make && make install
python -V

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```


#### 1.2 打印Python关键字
```
>>> import keyword
>>> print(keyword.kwlist)
['and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'not', 'or', 'pass', 'print', 'raise', 'return', 'try', 'while', 'with', 'yield']
>>> 
```
#### 1.3 Python 爬虫工具
```
HTTrack Website Copier
http://www.httrack.com/

```

### 2.  监控nginx端口状态并发送邮件
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2017/12/28 20:00
# @Author  : YangHongFei
# @File    : WebCheck.py

import os
import sys
import time
import smtplib
from email.mime.text import MIMEText
from email.MIMEMultipart import MIMEMultipart

def sendmail(warning):
    """
        功能：
            判断Nginx进程是否存在，失败则进行启动，成功后发送邮件通知用户确认！
            运行脚本时，可使用后台运行，默认定义5s检测一次，见参数：time.sleep(5)
        参数：
            time.sleep(5)
                默认每5s检测一次，若注释此参数将进入死循环默认
                若使用crontab进行运行脚本，可在while语句中加入break，注释此参数
        注意：
            若使用腾讯企业邮箱用户，并且开启二次认证，请使用授权码进行登录
            发信报错：535，腾讯用户参考链接：http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256
    """
    msg = MIMEText(warning)
    msg['Subject'] = 'Nginx Server monitor'
    msg['From'] = '别名随意定义'    #显示发件人别名，可进行修改
    try:
        smtp = smtplib.SMTP()
        smtp.connect(r'smtp.exmail.qq.com')
        smtp.login('发件人邮箱地址', '发件人邮箱密码')
        smtp.sendmail('收件人邮箱地址', ['收件人邮箱地址'], msg.as_string())
        smtp.close()
    except Exception, e:
        print e

while True:
     http_status = os.popen('netstat -tulnp | grep nginx', 'r').readlines()
     try:
         if http_status == []:
             os.system('/etc/init.d/nginx start')
             new_http_status = os.popen('netstat -tulnp | grep nginx', 'r').readlines()
             str1 = ''.join(new_http_status)
             is_80 = str1.split()[3].split(':')[-1]
             if is_80 != '80':
                 print 'Nginx 启动失败'
             else:
                 print 'Nginx 启动成功'
                 sendmail(warning="Nginx 异常关闭，启动成功，请确认！")  # 调用函数
         else:
             print 'Nginx is OK'
#             break
         time.sleep(5)
     except KeyboardInterrupt:
         sys.exit('\n')
```

### 3. Fabric装饰器远程登陆主机执行命令
```
#!/usr/bin/python
# -*- coding: utf-8 -*-

from fabric.api import *

env.use_ssh_config = True

env.hosts = {
    'gitlabusa' : ['172.16.0.20'],
    'gitlabbj' : ['172.16.0.176']
}
#env.roledefs = {
#    'gitlabusa' : ['172.16.0.20'],
#    'gitlabbj' : ['172.16.0.176']
#}
"""
    注意：为了安全起见，官方建议使用SSH-KEY方式进行连接服务器
    配置：将GitServer 公钥id_rsa.pud更新到GitProxy ~/.ssh/authorized_keys中
"""
env.user = 'root'
env.key_filename = '~/.ssh/id_rsa'


#@roles('*')
def mkdir():
    with cd("/tmp"):
        #run('mkdir yangxiaofei1111')
        run('ls /tmp')

#@roles('gitlabbj')
#def gitlabbj():
#    with cd("/tmp"):
#        run('mkdir yangxiaofei1111')
#
#def task():
#    execute(gitlabbj)
#    execute(gitlabusa)
```

### 4. asyncio 异步并发请求URL
```
import aiohttp
import asyncio
import time

# py -3 -m pip install aiohttp
# py -3 xxx.py
count = 0
async def request_get(url,num):
    async with aiohttp.ClientSession() as session:
        for _ in range(num):
            try:
                async with session.get(url) as response:
                    if response.status == 200:
                        global count
                        count +=1
                        print(count)
            except BaseException as e:
                print('error: %s'%e)

if __name__ == '__main__':
    start_time = time.time()
    url = 'https://www.qq.com/'
    loop = asyncio.get_event_loop()
    tasks = [request_get(url,1000)]
    loop.run_until_complete(asyncio.wait(tasks))
    print("the time : ",time.time() - start_time)
```
### 5. Python Django修改admin密码
```
进入到project目录，进入到Shell模式直接修改

python manage.py shell
from django.contrib.auth.models import User  
user =User.objects.get(username='admin') 
user.set_password('new_password')  
user.save()

如果超级用户也忘记了，尝试以下方式获取superuser
from django.contrib.auth.models import User
user1 = User.objects.filter(is_superuser = True)
user2 = User.objects.filter(is_superuser = True, is_staff = True) 
print user1, user2
```

### 6. Pycharm 激活
- http://idea.lanyus.com/
- https://www.jianshu.com/p/79df9ac88e96

### 7. GoogleAuth密钥生成
```python

import shortuuid
import base64

google_key = base64.b32encode(bytes(shortuuid.uuid() + shortuuid.uuid(), encoding="utf-8")).decode("utf-8")
print(google_key)
```

### 8. datetime转字符串
```python
from datetime import datetime
>>> end_time = datetime.datetime.now()
>>> start_time = end_time - datetime.timedelta(minutes=10)
>>> startTime = start_time.strftime("%Y-%m-%d %H:%M:%S")
>>> endTime = end_time.strftime("%Y-%m-%d %H:%M:%S")
>>> print(type(end_time))
<class 'datetime.datetime'>
>>> print(type(endTime))
<class 'str'>


// 判断是否是str，不是则返回时间--str
from datetime import datetime

def date_time_string(value):
    if isinstance(value,datetime):
        return value.strftime("%Y-%m-%d %H:%M:%S")
    else:
        return value

```

### 9. python不同路径导入模块
```python
import os
import sys
Base_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
sys.path.append(Base_DIR)
from yanghongfei.settings import ALY_INFO
```

### 10. python bytes-->dict
```python
import json
res = json.loads(response)
```

### 11. Python元组拆包

```python
>>> lax_coordinates = (33.9425, -118.408056)
>>> a,b = lax_coordinates
>>> a
33.9425
>>> b
-118.408056

#使用_
_ ,b = lax_coordinates

>>> b
-118.408056

>>> import os
>>> _, filename = os.path.split('/home/root/.ssh/id.pub')
>>> _
'/home/root/.ssh'
>>> filename
'id.pub'

```

