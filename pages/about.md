---
layout: page
title: About
description: 探索技术世界
keywords: Brian chen, 陈振洋
comments: true
menu: 关于
permalink: /about/
---

我是Brian，对技术的敬仰

对世界的向往，所要做的

便是勇往直前，发现更大

的世界

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
