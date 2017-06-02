---
layout: page
title: "热衷于开源项目，乐于钻研，闲余看门。"
tagline: "一个努力敲击键盘的码农（踏月寻玉）"
keywords: "博客,互联网,程序员"
description: "个人博客junerain.github.io"
---
{% include JB/setup %}

<p>&nbsp;</p>
{% for post in site.posts %}
  <h3><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3>
  <div style="color: #999;font-size: 14px;line-height: 1.6em;">{{ post.excerpt }}</div>
  <p>&nbsp;</p>
{% endfor %}
