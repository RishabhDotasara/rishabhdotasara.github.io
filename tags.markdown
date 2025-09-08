---
layout: page
title: Tags
permalink: /tags/
---

<div class="tags">
  {% assign tags = site.tags | sort %}
  {% for tag in tags %}
    <div class="tag-group">
      <h3 id="tag-{{ tag[0] | slugify }}">{{ tag[0] | capitalize }}</h3>
      <ul class="tag-posts">
        {% assign sorted_posts = tag[1] | sort: 'date' | reverse %}
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

{% if site.tags.size == 0 %}
  <p>No tags found yet. Start writing posts with tags!</p>
{% endif %}