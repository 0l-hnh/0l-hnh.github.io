---
layout: archive
permalink: /python/
title: "Python"

author_profile: true
sidebar:
  nav: "docs"
---

Python (tensroflow) / Pytorch

{% assign posts = site.categories.python %}
{% for post in posts %} {% include custom-archive-single.html type=page.entries_layout %} {% endfor %}
