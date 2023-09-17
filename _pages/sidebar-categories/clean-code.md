---
title: Clean Code
layout: archive
permalink: /clean-code
author_profile: true
types: posts
sidebar:
    nav: docs
---

{% assign posts = site.categories.clean-code %}
{% for post in posts %}
    {% include archive-single.html type=page.entries_layout %}
{% endfor %}