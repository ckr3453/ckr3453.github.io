---
title: "Book"
permalink: /categories/book
layout: archive
sidebar_category: true
---

{% assign posts = site.categories.book %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}