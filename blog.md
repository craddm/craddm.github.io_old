---
layout: page
title: Blog Archive
---

This year's posts

{% for post in site.posts %} {% unless post.next %}
{% else %} {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %} {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %} {% if year != nyear %}
{{ post.date | date: '%Y' }}

{% endif %} {% endunless %}
{{ post.date | date:"%d %b" }}: {{ post.title }}
{% endfor %}