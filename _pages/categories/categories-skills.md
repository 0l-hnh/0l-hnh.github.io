---
layout: archive
permalink: /skill/
title: "Skills"

author_profile: true
sidebar:
  nav: "docs"
---

문서 작성법, Tool 사용법 등 기술적 요소들

{% assign posts = site.categories.skill %}
{% for post in posts %} {% include custom-archive-single.html type=page.entries_layout %} {% endfor %}
