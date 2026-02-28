---
layout: default
title: Projects
---

# Projects

A collection of data analysis work spanning SQL, Python, and data visualization.
Each project includes the business question, methodology, and measurable results.

{% for project in site.projects %}
  {% include project-card.html project=project %}
{% endfor %}
