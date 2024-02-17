---
title: AWS
layout: archive
permalink: /aws
author_profile: true
types: posts
sidebar:
    - nav: "docs"
---

{% assign posts = site.tags.aws %}
{% for post in posts %}
    {% include archive-single.html type=page.entries_layout %}
{% endfor %}