---
title: Server
layout: archive
permalink: /server
author_profile: true
types: posts
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.server %}
{% for post in posts %}
    {% include archive-single.html type=page.entries_layout %}
{% endfor %}