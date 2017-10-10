---
layout: default
title: georg.io Explore articles by date
---

{% for post in site.posts %}
<li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
{{ post.content | remove: post.excerpt | strip_html | strip_newlines | truncatewords:75}}<br>
<a href="{{ post.url }}">Read more...</a><br><br>
{% endfor %}
