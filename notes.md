---
layout: page
title: Notes
comments: false
permalink: notes/
---
{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}
