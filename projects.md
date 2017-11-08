---
layout: page
title: Projects
permalink: /projects/index.html
---

This page lists a variety of projects that I have worked on. It is not an exhaustive list (some projects are not interesting enough to talk about, or not yet ready to
be talked about). Most of them have source code available, but at varying levels of quality (depending on my own level of experience at the time of working on them,
and depending on the nature of the project).

{% assign sorted = (site.projects | sort: 'date') | reverse %}
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
