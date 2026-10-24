---
layout: page
title: Projects
permalink: /projects/
description: Learning from patient data to understand cancer progression.
nav: true
nav_order: 3
---

<div class="projects">
  {% assign sorted_projects = site.projects | sort: "importance" %}
  {% for project in sorted_projects %}
    <a class="d-block mb-4" href="{{ project.redirect }}" aria-label="Read the preprint: {{ project.title | escape }}">
      <div class="card hoverable">
        <div class="card-body">
          <p class="text-muted mb-2">{{ project.publication_status }} &middot; {{ project.year }}</p>
          <h2 class="card-title">{{ project.title }}</h2>
          <p class="card-text">{{ project.description }}</p>
          <p class="card-text mb-0"><strong>Read the preprint &rarr;</strong></p>
        </div>
      </div>
    </a>
  {% endfor %}
</div>
