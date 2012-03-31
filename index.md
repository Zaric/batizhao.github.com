---
layout: page
title: bzz
tagline: 
---
{% include JB/setup %}

I'm **Bati Zhao**.

A web developer living in Shanghai, China. I currently work at **<a href="http://www.ideal.sh.cn/">Ideal</a>**
  coding **Java** apps. You can check some of my **code** and activity on **<a href="https://github.com/batizhao/">github</a>**. Feel free to **contact** me via **<a href="mailto:zhaobati@gmail.com">email</a>** or follow me on **<a href="https://twitter.com/batizhao">twitter</a>**.

<div id="homelinks">
  <div class="span-left code">
    <a href="https://github.com/batizhao/"><b>Code</b> → github.com/batizhao</a>
  </div>
  <div class="span-right last blog">
    <a href="http://batizhao.github.com">batizhao.github.com ← <b>Blog</b></a>
  </div>
  <div class="span-left email">
    <a href="mailto:zhaobati@gmail.com"><b>Email</b> → zhaobati@gmail.com</a>
  </div>
  <div class="span-right last twitter">
    <a href="https://twitter.com/batizhao">@batizhao ← <b>Twitter</b></a>
  </div>
</div>

<hr />

<ul class="posts" style="clear:both">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

