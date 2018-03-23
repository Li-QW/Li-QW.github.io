---
layout: page
title: About
description: 我是谁，我从哪里来……
keywords: 小我
comments: true
menu: 关于
permalink: /about/
---

未知就像游戏中的地图阴影。

那阴影既是恐惧的来源，也是认知的空缺。

越往外探索，半径越大，越是对新的探索力不从心。

沿着既定的方向摸索着前进，路越来越窄，队友都去了别路，只剩下孤独的自己。

回头看，已然找不到来时的路……

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
