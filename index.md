---
layout: page
title: CofCool的杂乱之语
---
{% include JB/setup %}

***沉沦于市井，已矣！***


本站记录了我学习计算机技术的过程中遇到的一些问题和如何这些问题的过程，一方面便于以后查看，另一方面也可以帮助和我遇到的一样问题的人们。还有一些没有什么用的杂乱之语，仅以慰藉罢了！


## 最近文章
<ul class="posts" style="margin: 0">
  {% for post in site.posts  limit:20 %}
    <li style="width:100%;height:110px;border-radius: 3px;box-shadow: 0px 1px 2px 0px rgba(0,0,0,0.15), 0px 2px 4px 0px rgba(0,0,0,0.10);border: 1px solid rgba(165,170,184,0.10);background: #FFFFFF;padding: 20px;transition: box-shadow 0.2s;-webkit-transition: box-shadow 0.2s;list-style:none;margin-bottom:10px">
      <span style="color:#A6A8B0;">{{ post.category }}</span>
      <h4><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h4>
      <div style="margin-bottom: 10px;color: gray;">
         {{ post.content | markdownify | strip_html | truncatewords: 50 }}
      </div>
      <div>
        {% for tag in post.tags %}
          <span style="border-radius: 6px;border: 1px solid gray;padding: 1px 4px;text-align: center;align-content: center;color:gray;font-size: small;display: inline-block;">
            <a href="/tag/{{ tag }}">{{ tag }}</a>
          </span> &nbsp;
        {% endfor %}
        <span style="float:right;color:gray;">{{ post.date | date_to_string }}</span>
      </div>
    </li>
  {% endfor %}
</ul>
