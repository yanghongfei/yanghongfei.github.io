---
layout: post
title: 监控 - 如何配置ZABBIX的Mail、SMS、Webhook的实现【附源码】
categories: ZABBIX
description: ZABBIX配置邮件、短信、webhook
keywords: ZABBIX
---


## 										记录下如何配置ZABBIX的Mail、SMS、Webhook的实现


### 前言

> 一般企业中监控ZABBIX还是最常见的，监控自然需要报警，那么报警最常见的比如一些邮件、短信，这样子出现问题了运维可以根据报警的信息，迅速进行解决问题，我也用ZABBIX很久了，前面文章里面也写了如果针对自己的业务需求实现[ZABBIX自定义模块](http://yanghongfei.me/2019/03/20/zabbix-template/) 自定义Key来实现监控， 本文章主要记录ZABBIX的通知配置及Webhooks怎么用，来帮助一些有需要的同学


**简单描述下我这面用到一些Mail SMS webhooks**

- Mail： 邮件告警，ZABBIX里面触发异常的Item默认都走邮件发出去，默认都会接入邮件通知
- SMS： 短信提醒，一般情况下，告警比较重要，高级别的报警可以接入短信
- Webhook: 可以理解为钩子，我是做故障自愈用的，将接收到的报警数据，进行分析匹配，执行一系列自定义任务，达到一些自愈的效果



### ZABBIX配置Mail

**Mailx插件**  

通过mailx 配置好外部SMTP服务器相关信息发送邮件

```shell

#install
yum install mailx -y

#配置
echo "set from=reportsender@domain.com smtp=smtp.exmail.qq.com smtp-auth-user=reportsender@domain.com smtp-auth-password=password smtp-auth=login" > /etc/mail.rc  

#发送测试
cat mail.txt | mailx -s "测试！" 1023671815@qq.com

```

**ZABBIX服务端配置**
- `/usr/lib/zabbix/alertscripts`

主要配置如下：  


```shell
$ sudo cat /etc/zabbix/zabbix_server.conf
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=NJflX4oJa68k
DBSocket=/var/lib/mysql/mysql.sock
SNMPTrapperFile=/var/log/snmptt/snmptt.log
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts

```

**Shell和Python的脚本内容你可以随意选择一个**

- `sendmail.sh`脚本内容

```shell
#!/bin/sh

export LANG=zh_CN.UTF-8
#$3是邮件内容 $2 邮件标题 $1 发送给谁
#echo "$3" | mail -s "$2" $1
FILE=/tmp/mailtmp.txt
echo "$3" >$FILE
dos2unix -k $FILE
/bin/mail -s "$2" $1 < $FILE

```

- `sendmail.py`脚本内容

```python
#!/usr/bin/env python
#coding:utf8
 
# 导入 smtplib 和 MIMEText 
import sys,getpass 
import smtplib 
from email.mime.text import MIMEText 

# 发送邮件函数 
def send_mail(to_list, sub,context): 
        me = mail_user + "<"+mail_user+"@"+mail_postfix+">" 
        msg = MIMEText(context,'plain','utf-8') 
        msg['Subject'] = sub 
        msg['From'] = me 
        msg['To'] = "".join(to_list) 
        try: 
                send_smtp = smtplib.SMTP() 
                send_smtp.connect(mail_host) 
                send_smtp.login(mail_user, mail_pass) 
                send_smtp.sendmail(me, to_list, msg.as_string()) 
                send_smtp.close() 
                return True 
        except Exception, e: 
                print str(e) 
                return False 

# 设置服务器名称、用户名、密码以及邮件后缀 
mail_host = "smtp.exmail.qq.com" 
mail_user = "reportsender@domain.com" 
mail_pass = "xxxxxxxxx" 
mail_postfix="exmail.qq.com" 

mailto_list = sys.argv[1] 
sub= sys.argv[2] 
context=sys.argv[3] 

if send_mail(mailto_list,sub,context): 
        print "Send mail succed!" 
else: 
        print "Send mail failed!"

```

提醒：配置完了，你可以执行脚本传三个参数测试下，不出错能发出去证明脚本没问题，又报错解决报错的问题。


**ZABBIX界面配置**

> 注意： ZABBIX3.x后配置`script`类型的时候就已经支持自定义传参了，但是2.x之前没有传参的选项

- ZABBIX 2.x示例

- 默认传输三个参数
  - 通知人
  - 告警标题
  - 告警内容  

- PATH: `Administer`-->`media_types`

![](/images/20190716094349.png)

- ZABBIX3.0+示例
- PATH: `Administer`-->`media_types`
- 参数：
  - {ALERT.SENDTO}
  - {ALERT.SUBJECT}
  - {ALERT.MESSAGE}


![](/images/20190716094501.png)

**配置Action(示例)**

**告警标题**

```

[自定义]{HOST.NAME}_{TRIGGER.STATUS}_{TRIGGER.SEVERITY}:{TRIGGER.NAME}:{HOST.IP}:{ITEM.KEY1}): {ITEM.VALUE1}

```

**告警内容**

```
[自定义]

告警主机: {HOST.NAME}
主机 IP : {HOST.IP}
告警时间: {EVENT.DATE} _ {EVENT.TIME}
告警状态: {TRIGGER.STATUS}
告警等级: {TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}

告警项目及值：

1. {ITEM.NAME1} : {ITEM.VALUE1}

```

**恢复标题**

```
[自定义]{HOST.NAME}_{TRIGGER.STATUS}_{TRIGGER.SEVERITY}:{TRIGGER.NAME}:{HOST.IP}:{ITEM.KEY1}): {ITEM.VALUE1}

```


**恢复内容**

```
[自定义]

告警主机: {HOST.NAME}
主机 IP : {HOST.IP}
告警时间: {EVENT.DATE}_{EVENT.TIME}
告警状态: {TRIGGER.STATUS}
告警等级: {TRIGGER.SEVERITY}
告警信息: {TRIGGER.NAME}

恢复项目及值：
 
1. {ITEM.NAME1} : {ITEM.VALUE1}

```

**示例图**

![](/images/20190716094651.png)



### ZABBIX配置SMS

- 这里SMS是基于阿里大鱼的，配置和邮件一样，就是通知方式不同，这里只附上源码脚本，不再进行一步步说明


- 脚本放到`/usr/lib/zabbix/alertscripts`
- 配置`media_types`
- 配置用户支持`SMS`
- 配置动作`Action`


```python  

import sys, json
from aliyunsdkdysmsapi.request.v20170525 import SendSmsRequest
from aliyunsdkcore.client import AcsClient
import uuid
from aliyunsdkcore.profile import region_provider

"""
pip install --upgrade setuptools
pip install aliyun-python-sdk-core
pip install aliyun-python-sdk-dysmsapi
"""
try:
    reload(sys)
    sys.setdefaultencoding('utf8')
except NameError:
    pass
except Exception as err:
    raise err

# 注意：不要更改
REGION = "cn-hangzhou"
PRODUCT_NAME = "Dysmsapi"
DOMAIN = "dysmsapi.aliyuncs.com"
ACCESS_KEY_ID = "xxxxxx"
ACCESS_KEY_SECRET = "xxxxxxxxx"

acs_client = AcsClient(ACCESS_KEY_ID, ACCESS_KEY_SECRET, REGION)
region_provider.add_endpoint(PRODUCT_NAME, REGION, DOMAIN)


def send_sms(business_id, phone_numbers, sign_name, template_code, template_param=None):
    sms_request = SendSmsRequest.SendSmsRequest()
    # 申请的短信模板编码,必填
    sms_request.set_TemplateCode(template_code)

    # 短信模板变量参数
    if template_param is not None:
        sms_request.set_TemplateParam(template_param)

    # 设置业务请求流水号，必填。
    sms_request.set_OutId(business_id)

    # 短信签名
    sms_request.set_SignName(sign_name)

    # 短信发送的号码列表，必填。
    sms_request.set_PhoneNumbers(phone_numbers)

    # 调用短信发送接口，返回json
    sms_response = acs_client.do_action_with_exception(sms_request)

    ##业务处理
    return sms_response


if __name__ == '__main__':
    __business_id = uuid.uuid1()
    sendto = str(sys.argv[1])  # zabbix传过来的第一个参数
    message = str(sys.argv[2])  # zabbix传过来的第二个参数
    params = {"msg": message}  # 对应短信模板里设置的参数
    params = json.dumps(params)
    print(send_sms(__business_id, sendto, "阿里大鱼查看", "阿里大鱼查看", params))
```



### ZABBIX配置Webhooks

> 配置webhooks最主要是为了接收ZABBIX触发的告警，POST发送到你自定义接口中，不但可以实现故障恢复，也可以根据报警分析等操作，具体看个人需求， 这里只介绍ZABBIX3.x+的版本


这些告警信息知识你需要了解

- {ALERT.SUBJECT}： 告警主机的主题
- {HOST.NAME} ： 告警主机名称
- {HOST.IP} ： 告警主机IP
- {TRIGGER.NAME} ： 告警主机触发器名字
- {TRIGGER.SEVERITY}： 告警主机触发级别
- {TRIGGER.STATUS} ： 告警主机触发状态 


**首先你需要有一个接口**

这里我简单用`tornado`模拟了一个接口

```python
import tornado.web

class ZabbixSendAlertHandler(BaseHandler):

    def post(self, *args, **kwargs):
        data = json.loads(self.request.body.decode("utf-8"))
        print(data)
        return self.write(dict(data=data))


#这里自己封装了，不用关心
zabbix_urls = [
    (r"/v1/zabbix/send_alert/", ZabbixSendAlertHandler),
]

```

**其次你需要一个向接口提交数据的脚本**

因为我的接口最终是根据[OpenDevOps运维自动化平台](https://github.com/opendevops-cn/opendevops)使用的,所以我提供的脚本里面也带了认证

脚本放在那？  这个要看你配置在那里了，不管是Mail\SMS\Webhooks  只要是触发脚本类型的都在这里

```shell
#让我们查一下你的配置是什么？

$ grep -r "AlertScriptsPath" /etc/zabbix/zabbix_server.conf

AlertScriptsPath=/usr/lib/zabbix/alertscripts   #脚本就放在这里

```

示例脚本：

```python

#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/7/15 10:09
# @Author  : Fred Yangxiaofei
# @File    : send_alert.py
# @Role    : Zabbix webhooks 发送告警信息


import sys
import requests
import json

def send_alert(webhook_url, messages):
    params = {
        "auth_key": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.这个Key是我平台自己申请的"
    }
    # payload = "{\"data\": \"test\"}"
    payload = {"data": messages}
    headers = {'content-type': "application/json", 'cache-control': "no-cache"}
    response = requests.post(url=webhook_url, data=json.dumps(payload), headers=headers, params=params)
    print(response.text)



if __name__ == '__main__':
    webhook_url = sys.argv[1]
    messages = sys.argv[2]
    send_alert(webhook_url, messages)

"""
如何使用：

pip install json
pip install requests

如何测试：

保证脚本没问题，你先手动执行下看看
python send_alert_to_codo.py http://172.16.0.101:8040/v1/zabbix/send_alert/ alert_message_test

```

这里你可以看到我脚本需要2个参数，接下来就是我们要配置的，从ZABBIX里面传参给到我的脚本

**接下来我们配置ZABBIX**

- PATH: `Administer`-->`media_types`

![](/images/20190716100641.png)


- PATH：`Configuration`-->`Action`
- 为了不影响你现有的`Action`我们专门创建一个Action来触发我们的Webhooks消息

标题内容，这里需要全面一点，我给关键信息都放进去了，然后三个下划线分开的，后续我要做故障自愈用

```
{HOSTNAME}___{HOST.IP}___{TRIGGER.NAME}___{TRIGGER.STATUS}___{TRIGGER.SEVERITY}

```

- 新`clone`一个`Action`

![](/images/20190716100752.png)

- 主要配置标题(记住这里告警和恢复都设置)、发送类型、发送人

![](/images/20190716101054.png)

- 选择上图Action指定的用户，一定让这个User也接收SendAlert的报警才可以发出去

![](/images/20190716101959.png)


**最后，我们模拟一个故障看我的接口是否可以接受到我的告警信息**

- 这里我停了自己的Agent 然后又启动了起来，可以看到如下图我可以接收到信息了

- Authkey： 是我自己的接口验证，自然要马赛克下哈。

```shell
#告警信息
{'data': 'Zabbix server___127.0.0.1___Zabbix agent on Zabbix server is unreachable for 5 minutes___PROBLEM___Average'}
[I 190716 10:27:32 web:2106] 200 POST /v1/zabbix/send_alert/?auth_key=xxxx
#恢复信息
{'data': 'Zabbix server___127.0.0.1___Zabbix agent on Zabbix server is unreachable for 5 minutes___OK___Average'}
[I 190716 10:30:09 web:2106] 200 POST /v1/zabbix/send_alert/?auth_key=xxxx

```


![](/images/20190716103413.png)










