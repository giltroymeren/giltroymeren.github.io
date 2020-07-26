---
layout: page
permalink: /categories/
title: Categories
---


<div id="archives">
{% assign alphabetical_categories = site.categories | sort %}
{% for category in alphabetical_categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>

    <a name="{{ category_name | slugize }}"></a>
    <h3 class="category-head">{{ category_name }}</h3>

    <ul class="archive-item">
    {% assign alphabetical_posts = site.categories[category_name] | sort: "title" %}
    {% for post in alphabetical_posts %}

        <li>
            <a href="{{ site.baseurl }}{{ post.url }}">
                {% if post.title and post.title != "" %}
                    {{post.title}}
                {% else %}
                    {{post.excerpt |strip_html}}
                {%endif%}
            </a>
        </li>
    {% endfor %}
    </ul>
  </div>
{% endfor %}
</div>