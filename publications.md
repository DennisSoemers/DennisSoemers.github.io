---
layout: page
title: Publications
permalink: /publications/
---

This page lists all of my scientific publications. They are sorted by publication type, and within each publication type sorted in the reverse order of publication date.

{% for static_file in site.static_files %}
  {% if static_file.path contains 'publications_text' %}
  **Last updated:** {{ static_file.modified_time | date: "%d-%b-%y" }}.
  {% include_relative {{ static_file.path }} %}
  {% endif %}
{% endfor %}
