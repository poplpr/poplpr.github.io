---
layout: page
title: About
description: 打码改变世界
keywords: poplpr
comments: true
menu: 关于
permalink: /about/
---

我是poplpr，目前失业中。

曾经拿到过 icpc/ccpc Ag\*4，蓝桥杯国一\*4.

现在正在赶 NLP 论文，但是可惜的是我在学术上完全没有天赋，期望赶完论文赶紧向工程转型。

在工程上啥也不会……很糟糕的校招……

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ site.url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
