---
layout:     post
title:      "GitLab API Enable Depoly Keys For Project"
subtitle:   " \"GitLab API开启Depoly Key\""
date:       2019-03-19
author:     "Yangxiaofei"
header-img: "img/gitlab.jpg"
tags:
    - GitLab
---

## 使用GitLab API为指定项目开启DepolyKeys

### 背景
- 为什么使用DepolyKeys?
    - 1. 安全
    - 2. 不需要像SSH Keys一样开通用户
    - 3. Depoly Keys是全局的，每个项目都可以enable
    - 4. Depoly Keys不需要担心用户离职删除账户问题
    - 5. Depoly Keys 只有`git pull`的权限，没有`git push`权限，用于发布足够安全


- 我们这里发布流程大概是这样子:
    - 1. gitlab提交代码，打Tag出发update钩子
    - 2. 钩子逻辑匹配，像运维平台提交对应发布任务
    - 3. 运维平台连接指定SaltServer服务器进行代码分发操作
    - 4. 代码发布：`enable depoly keys``git clone`-->`git pull`--->`git checkout <tag>`--->`rsync_code`--->`code_backup`--->`publish code`从而完成整个操作

### 实现

- 官方文档：https://docs.gitlab.com/ee/api/deploy_keys.html
> 以上问题，除了每次都要登陆gitlab找到对应的版本库信息进行`Enabled DepolyKeys`外，所有的都已经全自动化操作，懒顿即美德，所以想着Gitlab既然有API，那就用API来让他自动操作吧，所以本章主要记录下GitLab API 开启 Depoly Keys的过程。

### 授权
#### GitLab生成Access Token
> 既然要用APi，认证信息必然不可少。

- 我们登陆GitLab 找到账户信息，生成一个Token，登陆进入Profile Setting — Account

![image](https://ws1.sinaimg.cn/large/005X1wn0gy1g1892l6ru6j31gb0nrju1.jpg)

![image](https://ws1.sinaimg.cn/large/005X1wn0gy1g1892l4gsuj31gj0hcac6.jpg)

- API使用

```shell
#get就可以列出所有的keys
GET /deploy_keys
curl --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/deploy_keys"
```

![image](https://ws1.sinaimg.cn/large/005X1wn0gy1g1892l68t0j31gy054gpw.jpg)


#### 简单了解下API用法，直接上我们用到的`Enable Depoly key`

官方链接：https://docs.gitlab.com/ee/api/deploy_keys.html#enable-a-deploy-key
![](https://ws1.sinaimg.cn/large/005X1wn0gy1g189dz1rm7j30vr0hodgt.jpg)

以上，开启需要知道你的Depoly KeysID和项目ID
```shell

curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" https://gitlab.example.com/api/v4/projects/5/deploy_keys/13/enable
```
以下是我的python脚本实例，仅供参考


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2019/3/19 14:06
# @Author  : Fred Yangxiaofei
# @File    : depoly_key_enable.py
# @Role    : 通过API启用Deploy_key

import os
import sys

Base_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(Base_DIR)

from yanghongfei.settings import GITLAB_TOKEN
import json
import requests
import fire


class GitLabAPI():
    def __init__(self, project_name):
        """
        :param project_name:  项目名字，简写，如：gitlab里面的项目名字
        """
        self.access_token = GITLAB_TOKEN
        self.gitlab_url = 'http://doamin.com'
        self.project_name = project_name

    def get_project_id(self):
        project_url = "{}/api/v4/projects/".format(self.gitlab_url)
        headers = {
            "PRIVATE-TOKEN": self.access_token
        }
        resp = requests.get(project_url, headers=headers)
        if resp.status_code == 200:
            projects_info = json.loads(resp.text)
            for project in projects_info:
                if self.project_name in project['name']:
                    return project['id']
        else:
            print('[Error]: GitlabALy访问失败！')
            exit(-1)

    def enable_project_depoly_keys(self):
        project_id = self.get_project_id()
        depoly_key_id = '4'  # 我们只有一个id，就不写脚本获取了，直接写在这里，获取方式：curl --header "PRIVATE-TOKEN: <your_access_token>" "http://domain.com/api/v4/deploy_keys"
        project_url = "{}/api/v4/projects/{}/deploy_keys/{}/enable".format(self.gitlab_url, project_id, depoly_key_id)
        headers = {
            "PRIVATE-TOKEN": self.access_token
        }
        resp = requests.post(project_url, headers=headers)
        if resp.status_code == 201:
            print('[Sucess]: {} DepolyKeys Enable sucessfully!'.format(self.project_name))
        else:
            print('[Error]: {} DepolyKeys Enable falied!'.format(self.project_name))
            exit(-2)


def main(project_name):
    obj = GitLabAPI(project_name)
    obj.enable_project_depoly_keys()


if __name__ == '__main__':
    fire.Fire(main)

```
