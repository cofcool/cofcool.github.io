---
layout: page
title: CofCool的杂乱之语
---
{% include JB/setup %}

***沉沦于市井，已矣！***


本站记录了我学习计算机技术的过程中遇到的一些问题和如何这些问题的过程，一方面便于以后查看，另一方面也可以帮助和我遇到的一样问题的人们。还有一些没有什么用的杂乱之语，仅以慰藉罢了！


## 最近文章
<ul class="posts" style="margin: 0">
  {% for post in site.posts  limit:15 %}
    <li style="width:100%;height:140px;border-radius: 3px;box-shadow: 0px 1px 2px 0px rgba(0,0,0,0.15), 0px 2px 4px 0px rgba(0,0,0,0.10);border: 1px solid rgba(165,170,184,0.10);background: #FFFFFF;padding: 10px;transition: box-shadow 0.2s;-webkit-transition: box-shadow 0.2s;list-style:none;margin-bottom:10px">
      <span>
        <a style="color:#A6A8B0;" href="/categories.html#{{ post.category }}-ref">{{ post.category }}</a>
      </span>
      <h4><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h4>
      <div style="margin-bottom: 15px;color: gray;height: 40px;">
         {{ post.excerpt | remove: '<p>' | remove: '</p>' | strip_html }}
      </div>
      <div style="position: relative">
        <ul class="tag_box inline">
          <li><i class="icon-tags"></i>&nbsp;</li>
          {% for tag in post.tags %}
            <li>
              <a href="/tags.html#{{ tag }}-ref">{{ tag }}</a>
            </li>
          {% endfor %}
        </ul>
        <span style="float:right;color:gray;position: absolute;right: 0;top: 5px;">{{ post.date | date_to_string }}</span>
      </div>
    </li>
  {% endfor %}
</ul>
