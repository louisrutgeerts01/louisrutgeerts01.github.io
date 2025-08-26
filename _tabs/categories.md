---
layout: page
title: Categories
permalink: /categories/
icon: fas fa-stream
order: 1
---

# Categories

Browse posts grouped by category:

{% for category in site.categories %}
  <h2 id="{{ category[0] }}">{{ category[0] | capitalize }}</h2>
  <ul>
  {% for post in category[1] %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
{% endfor %}