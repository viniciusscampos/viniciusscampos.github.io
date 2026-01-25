---
layout: default
title: Home
---

<section class="posts">
  <h2 class="section-title">Writing</h2>
  <ul class="post-list">
    {% for post in site.posts %}
      <li class="post-list-item">
        <span class="post-list-date">{{ post.date | date: "%b %d, %Y" }}</span>
        <a class="post-list-title" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        {% if post.description %}
          <p class="post-list-description">{{ post.description }}</p>
        {% endif %}
      </li>
    {% endfor %}
  </ul>
</section>
