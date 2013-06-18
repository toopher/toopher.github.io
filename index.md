---
layout: page
title: Toopher Dev Blog
tagline: Engineering the Second Factor
---
{% include JB/setup %}

{% include JB/pages_list %}

[Toopher](https://www.toopher.com/) is a security startup based in sunny
Austin, TX. We're a polyglot dev shop working on big challenges, like
making security usable.

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
