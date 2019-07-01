---
layout: page
title: About
description: 一无所知的世界走下去才会有惊喜！
keywords: I'm Yangxiaofei
comments: true
menu: 关于
permalink: /about/
---

Hi, I'm Yangxiaofei

游戏行业，运维工程师

写博客不单单是为了记录，为了资源分享

当你返回来看的时候，你会发现这会是很有意思的一件事！


## Contact


{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## About Work

- [HTU](http://htuidc.com)
- [Shinezone](http://www.shinezone.com)

<!-- ## Skill Keywords

- Linux
- Shell
- Python
- Tornado
- Vue
- iview-admin -->

## A Simeple info  

- Name,Fred Yang(YangHongFei)
- From,ShangQiu Henan
- Now, Pudong Shanghai
- Wok in [Shinezone Network Co., Ltd.](http://www.shinezone.com)
- Operation and Maintenance Engineer
- Here you can see before [My Blog](http://www.cnblogs.com/yangxiaofei)



## Skill Keywords

> 平时工作中接触到的一些技术

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
