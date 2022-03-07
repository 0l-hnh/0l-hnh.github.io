---
layout: archive
permalink: /os/
title: "Os"

author_profile: true
sidebar:
  nav: "docs"
---

Windows / Linux (RedHat, Ubuntu)

{% assign posts = site.categories.os %}
{% for post in posts %} {% include custom-archive-single.html type=page.entries_layout %} {% endfor %}
