---
layout: default
title: georg.io - Numerical Analysis
---

{% for post in site.posts %}
{% for tag in post.tags %}
{% if tag == "Numerical-Analysis" %}
<li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
{{ post.content | strip_html | truncatewords:75}}<br>
<a href="{{ post.url }}">Read more...</a><br><br>
{% endif %}
{% endfor %}
{% endfor %}
