---
layout: page
title: CofCool的杂乱之语
---
{% include JB/setup %}

***沉沦于市井，已矣！***


本站记录了我学习计算机技术的过程中遇到的一些问题和如何这些问题的过程，一方面便于以后查看，另一方面也可以帮助和我遇到的一样问题的人们。还有一些没有什么用的杂乱之语，仅以慰藉罢了！


## 最近文章
<ul class="posts">
  {% for post in site.posts  limit:20 %}
    <li>
      <span style="color:'#A6A8B0';">{{ post.category }}</span>
      <h4><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h4>
      <div>
        {% for tag in post.tags %}
          <span>{{ post.tag }}</span> &nbsp;
        {% endfor %}
        <span style="float:right;">{{ post.date | date_to_string }}</span>
      </div>
    </li>
  {% endfor %}
</ul>
