---
layout: page
title: 主页
tagline: 奉至修真行，唯有逆者！！！
---
{% include JB/setup %}
## 关于我

***生活，不只是活着！沉沦于市井，已矣！***

本站记录了我学习计算机技术的过程中遇到的一些问题和如何这些问题的过程，一方面便于以后查看，另一方面也可以帮助和我遇到的一样问题的人们。还有一些没有什么用的杂乱之语，仅以慰藉罢了！

<a href="http://i.youku.com/cofcool" target="_blank">我的优酷主页</a>

## 最近文章
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## 联系我

邮箱地址:<cofcool@126.com>




