---
layout: default
title: Posts
---

<ul class="post-list">
{% for post in site.posts %}
  <li>
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="date">{{ post.date | date: "%-d %b %Y" }}</p>
    <p class="excerpt">{{ post.excerpt | strip_html | strip }}</p>
  </li>
{% endfor %}
</ul>
