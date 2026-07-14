---
title: "Projects"
layout: archive
permalink: /portfolio/
author_profile: true
---

{% assign portfolio_items = site.portfolio | sort: 'date' | reverse %}
{% for post in portfolio_items %}
  {% include archive-single.html type="grid" %}
{% endfor %}
