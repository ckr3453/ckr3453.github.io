---
title: "categorytest"
permalink: /categories/categorytest
layout: archive
sidebar_category: true
---

{% assign posts = site.categories.categorytest %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}