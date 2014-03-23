---
layout: page
title: Toopher Dev Blog
tagline: Engineering the Second Factor
---
{% include JB/setup %}

{% include JB/pages_list %}

## Toopher

[Toopher](https://www.toopher.com/) is a security startup based in sunny
Austin, TX. We're a polyglot dev shop making security usable. We do cool
stuff and have fun, and [we're hiring](https://toopher.com/hiring).

![Toopher logo](/assets/images/toopher-logo.png)

Looking for help with something technical? Try the [Toopher
docs](https://dev.toopher.com/).

### Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{% if post.display_title %}{{ post.display_title }}{% else %}{{ post.title }}{% endif %}</a></li>
  {% endfor %}
</ul>
