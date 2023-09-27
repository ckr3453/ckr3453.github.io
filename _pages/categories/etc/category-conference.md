---
title: "Conference/Seminar"
permalink: /categories/conference
layout: archive
sidebar_category: true
---

{% assign posts = site.categories.conference %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}