---
layout: default
title: Categories
permalink: /categories/
---

# Categories

Below are all blog categories on this site. Click any category to view related posts.

{% for category in site.categories %}

## [{{ category[0] | capitalize }}](/category/{{ category[0] | slugify }}/)

{% assign posts = category[1] %}
<ul>
  {% for post in posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span style="color:#888; font-size:0.9em;"> — {{ post.date | date: "%b %-d, %Y" }}</span>
  </li>
  {% endfor %}
</ul>

{% endfor %}