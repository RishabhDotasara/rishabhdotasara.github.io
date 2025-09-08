---
layout: page
title: Categories
permalink: /categories/
---

<div class="categories">
  {% assign categories = site.categories | sort %}
  {% for category in categories %}
    <div class="category-group">
      <h3 id="{{ category[0] | slugify }}">{{ category[0] | capitalize }}</h3>
      <ul class="category-posts">
        {% assign sorted_posts = category[1] | sort: 'date' | reverse %}
        {% for post in sorted_posts %}
          <li>
            <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
            <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
          </li>
        {% endfor %}
      </ul>
    </div>
  {% endfor %}
</div>

{% if site.categories.size == 0 %}
  <p>No categories found yet. Start writing posts with categories!</p>
{% endif %}