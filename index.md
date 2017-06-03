---
layout: page
title: "后端,C++,UNIX,云计算"
tagline: "一个努力敲击键盘的码农（踏月寻玉）"
keywords: "后端,C++,UNIX,云计算"
description: "个人博客junerain.github.io"
---
{% include JB/setup %}

<p>&nbsp;</p>
{% for post in site.posts %}
  <h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3>
  <div style="color: #999;font-size: 14px;line-height: 1.6em;">{{ post.excerpt }}</div>
  <p>&nbsp;</p>
{% endfor %}
