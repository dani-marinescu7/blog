---
layout: home
title: Welcome to my blog!
---

# Latest Posts

{% for post in site.posts %}
  <article class="post-preview">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</p>
    <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 50 }}</p>
  </article>
{% endfor %}
