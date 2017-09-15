---
layout: page
permalink: /xamep/
title: Теория всего
---

{% for post in site.categories.xamep %}
 <li><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
