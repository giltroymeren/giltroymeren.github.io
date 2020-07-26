---
layout: page
title: Categories
permalink: /categories/
---

{% assign alphabetical_categories = site.categories | sort %}
<ul>
{% for category in alphabetical_categories %}
    <li><a href="{{ category[0] }}">{{ category[0] | escape }}</a></li>
{% endfor %}
</ul>
