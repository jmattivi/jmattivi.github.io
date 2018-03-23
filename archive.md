---
layout: splash
title: "Archive"
permalink: /archive/
header:
        image: /assets/images/banner2.png
---

{% for post in site.posts %} {% unless post.draft %}

{{ post.date | date_to_string }} » [ {{ post.title }} ]({{ post.url }}) {% endunless %}
{% endfor %}