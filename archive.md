---
layout: page
title: "Archive"
permalink: /archive/
---

{% for post in site.posts %} {% unless post.draft %}

{{ post.date | date_to_string }} » [ {{ post.title }} ]({{ post.url }}) {% endunless %}
{% endfor %}