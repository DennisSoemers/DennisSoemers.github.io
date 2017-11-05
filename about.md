---
layout: page
title: About
permalink: /about/
---

{% for static_file in site.static_files %}
  {% if static_file.path contains 'about_text' %}
  {% include_relative {{ static_file.path }} %}
	
  Last updated: {{ static_file.modified_time | date: "%d-%b-%y" }}.
  {% endif %}
{% endfor %}