---
layout: page
title: About
description: Hey, this is PZ
keywords: Panzi, 胡畔 , hupan
comments: true
menu: 关于
permalink: /about/
---

Hey，我是PZ， 一只ITer，现在WH工作

仰慕「优雅编码的艺术」。

工作、学习之余，我还是一个科幻爱好者。

生活不止眼前苟且，还有远方的苟且

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
