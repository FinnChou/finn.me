---
layout: page
title: Blog archive
permalink: /archive/
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="..{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>


