---
layout: page
title: Portfolio
permalink: /portfolio/
---

#### Portfolio

Projects, publications and other stuff I've worked on.

---

## Publications and Conferences
<div class="portfolio-items">
  {% for project in site.data.publications %}
  <div class="row" style="margin-top: 7%">
    <div class="three columns">
      <a href="{{ project.url }}" target="_blank"><img src="{{ site.baseurl }}/assets/images/projects/{{ project.thumbnail }}" class="portfolio-thumbnail"></a>
    </div>
    <div class="nine columns">
      <span class="project-authors"><i>{{ project.authors }}</i></span><br>
      <span class="project-title">{{ project.title }}</span><br>
      <span class="project-description">{{ project.description }}</span><br>
      <span class="project-url"><b><a href="{{ project.url }}" target="_blank">{{ project.journal }}.</a></b> {{ project.date }}</span>
    </div>
  </div>
  {% endfor %}
</div>

---

## Projects
<div class="portfolio-items">
  {% for project in site.data.projects %}
  <div class="row" style="margin-top: 7%">
    <div class="three columns">
      <a href="{{ project.url }}" target="_blank"><img src="{{ site.baseurl }}/assets/images/projects/{{ project.thumbnail }}" class="portfolio-thumbnail"></a>
    </div>
    <div class="nine columns">
      <span class="project-authors"><i>{{ project.authors }}</i></span><br>
      <span class="project-title">{{ project.title }}</span><br>
      <span class="project-description">{{ project.description }}</span><br>
      <span class="project-url"><b><a href="{{ project.url }}" target="_blank">{{ project.journal }}.</a></b> {{ project.date }}</span>
    </div>
  </div>
  {% endfor %}
</div>