---
layout: page
title: Portfolio
permalink: /portfolio/
---

#### Portfolio

Projects, publications and other stuff I've worked on.

<div class="portfolio-items">
  {% for project in site.data.projects %}
  <div class="row" style="margin-top: 7%">
    <div class="three columns">
      <a href="{{ project.url }}" target="_blank"><img src="{{ site.baseurl }}/assets/images/projects/{{ project.thumbnail }}" class="portfolio-thumbnail"></a>
    </div>
    <div class="nine columns">
      <p class="no-btm-margin"><i>{{ project.authors }}</i></p>
      <h3>{{ project.title }}</h3>
      <!-- <p>{{ project.description }}</p> -->
      <span class="lighter"><b><a href="{{ project.url }}" target="_blank">{{ project.journal }}.</a></b> {{ project.date }}</span>
    </div>
  </div>
  {% endfor %}
</div>