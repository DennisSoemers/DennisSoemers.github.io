---
layout: page
title: Projects
permalink: /projects/index.html
description: A list of projects I have worked on / am working on.
---

This page lists a variety of projects that I have worked on. It is not an exhaustive list (some projects are not interesting enough to talk about, or not yet ready to
be talked about). Most of them have source code available and/or publications (papers, theses or other kinds of reports) available, but not all.

{% assign sorted = site.projects | sort: 'date' | reverse %}
{% for item in sorted %}
  {% if item.image %}
  <a href="{{ item.url }}"><img src="{{ item.image }}" align="right" style="width:256px"></a>
  {% endif %}
  <h2>
    <a class="post-link" href="{{ item.url }}">{{ item.title }}</a>
  </h2>
  {{ item.description }}
  
  <a class="post-meta" href="{{ item.url }}">Read more...</a>
{% endfor %}
