---
layout: page
title: Categories
permalink: /categories/
---

<div class="categories">
  {% assign categories = site.categories | sort %}
  {% for category in categories %}
    <div class="category-group">
      <h2 id="{{ category[0] | slugify }}">
        <a href="/categories/{{ category[0] | slugify }}/">{{ category[0] | capitalize }}</a>
      </h2>
      <p>{{ category[1] | size }} post{% if category[1].size != 1 %}s{% endif %}</p>
    </div>
  {% endfor %}
</div>