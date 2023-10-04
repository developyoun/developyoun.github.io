---
title: Design Pattern
layout: archive
permalink: /design-pattern
author_profile: true
types: posts
sidebar:
    nav: docs
---

{% assign posts = site.categories["디자인패턴"] %}
{% for post in posts %}
    {% include archive-single.html type=page.entries_layout %}
{% endfor %}