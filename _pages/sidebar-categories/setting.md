---
title: Settings
layout: archive
permalink: /settings
author_profile: true
types: posts
sidebar:
    - nav: "docs"
---

{% assign posts = site.tags.settings %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}