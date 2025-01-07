---
layout: page
title: Blog archive
permalink: /archive/
---

{% assign tags = site.posts | map: "tags" | uniq %}
{% for tag in tags %}
<h3>{{ tag }}</h3>
<p>
{% case tag %}
{% when "数字信号处理" %}
描述 1111
{% when "数字图像处理" %}
描述 2222
{% endcase %}
</p>
<hr style="border: 2px solid #ccc; margin: 20px 0;">
<ul>
  {% for post in site.posts %}
    {% if post.tags contains tag %}
    <li>
      <a href="..{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endif %}
  {% endfor %}
</ul>
<br>
{% endfor %}

