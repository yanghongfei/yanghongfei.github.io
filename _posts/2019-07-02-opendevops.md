---
layout: post
title: 开源推荐 - CoDo开源一站式DevOps平台
categories: DevOps
description: 开源推荐 - CoDo开源一站式DevOps平台
keywords: opendevops, devops 
---

> 一群有梦想的年轻人开源了一个云管理平台，他们的口号是：让天下没有996的运维

# 开源推荐 - CoDo开源一站式DevOps平台

有幸参与到CoDo项目的开发，这是一个非常棒的一站式开源运维平台，分享给大家



## 平台介绍

CODO是一款为用户提供企业多混合云、自动化运维、完全开源的云管理平台。

CODO前端基于Vue iview开发、为用户提供友好的操作界面，增强用户体验。

CODO后端基于Python Tornado开发，其优势为轻量、简洁清晰、异步非阻塞。

CODO开源多云管理平台将为用户提供多功能：ITSM、基于RBAC权限系统、Web Terminnal登陆日志审计、录像回放、强大的作业调度系统、CMDB、监控报警系统、DNS管理、配置中心等

## 产品架构

![](https://blz.nos-hz.163yun.com/sre/images/20190530.codo.01.png)

## 产品功能  

![](https://blz.nos-hz.163yun.com/sre/images/20190530.codo.02.png)

## 模块说明

- 项目前端：基于Vue + Iview-Admin实现的一套后台管理系统

- 管理后端：基于Tornado实现，提供Restful风格的API，提供基于RBAC的完善权限管理，可对所有用户的操作进行审计

- 定时任务：基于Tornado实现，定时任务系统，完全兼容Linux Crontab语法，且支持到秒级

- 任务调度：基于Tornado实现，系统核心调度，可分布式扩展，自由编排任务，自由定义流程，支持多种触发，支持审批审核，支持操作干预

- 资产管理：基于Tornado实现，资产管理系统，支持手动添加资产，同时也支持从AWS/阿里云/腾讯云自动获取资产信息

-  配置中心：基于Tornado实现，可基于不同项目、环境管理配置，支持语法高亮、历史版本差异对比、快速回滚，并提供Restful风格的API

- 域名管理：基于Tornado实现，支持多区域智能解析、可视化Bind操作、操作日志记录

- 运维工具：基于Tornado实现，运维场景中常用的加密解密、事件、故障、项目记录、提醒、报警等


## 在线体验  

CoDo提供了在线Demo供使用者体验，Demo账号只有部分权限

- 地址：http://demo.opendevops.cn
- 用户：demo
- 密码：2ZbFYNv9WibWcR7GB6kcEY  

![](https://blz.nos-hz.163yun.com/sre/images/20190530.codo.03.png)

## 推荐理由

**团队出品**： github上有很多开源的devops工具，几乎全部都由个人发布维护，代码质量、版本进度以及可持续性都无法保障，陷入不能用或不敢用的尴尬境地，CoDo非个人项目，由一个团队负责开发维护，有幸我也是团队中一员，参与贡献了部分代码，所以在稳定性和持续性方面更有保证

**生产实践**： CoDo核心代码贡献者全部来自于一线运维团队，团队成员从运维需求出发，致力于解决运维痛点，更了解运维的需求，且核心代码经过了多年生产实践，并非实验产品，运行稳定

**功能齐全**： CoDo采用微服务的理念构建，模块化开发，目前已有资产管理、定时任务、任务调度、配置中心、域名管理、运维工具几大模块，支持持续集成、持续部署、代码审查、数据库审核与优化建议等众多功能，覆盖大部分的运维场景，让你不再费心劳神在多个系统间奔波，一个平台全搞定

**完善支持**： CoDo除了提供专业的文档支持外，还同时开始录制一些基础的部署使用视频帮助初学者快速上手，如果你觉得这些还不够，我们也提供QQ或微信远程支持，助你顺利部署使用

**开源免费**： 这是一个开源项目，所有功能均可免费使用，源码托管在GitHub


## 项目地址

官网：http://www.opendevops.cn

GitHub：https://github.com/opendevops-cn

文档地址：http://docs.opendevops.cn/zh/latest

安装视频：https://www.bilibili.com/video/av53446517


最后欢迎大家使用，如有任何意见和建议都可以通过ISSUE或者QQ群反馈给我们，我们会进行持续的更新和优化

  

转载来自：[运维咖啡吧](https://ops-coffee.cn/s/eZX5qoJzuCSuLI98gSCtpw)

![](http://blz.nos.netease.com/sre/oa.qrcode.png)