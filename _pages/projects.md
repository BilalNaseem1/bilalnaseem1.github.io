---
layout: page
title: Portfolio
permalink: /projects/
description: A collection of data engineering and machine learning projects — pipelines, platforms, and production systems.
nav: true
nav_order: 2
horizontal: false
---

<div class="projects">
  {%- assign all_tags = "" | split: "" -%}
  {%- for project in site.projects -%}
    {%- for tag in project.tags -%}
      {%- unless all_tags contains tag -%}
        {%- assign all_tags = all_tags | push: tag -%}
      {%- endunless -%}
    {%- endfor -%}
  {%- endfor -%}
  {%- assign all_tags = all_tags | sort -%}

  {%- if all_tags.size > 0 %}
  <div class="tag-filter-bar">
    <span class="filter-label">Filter:</span>
    {%- for tag in all_tags %}
    <button class="tag-filter-btn" data-tag="{{ tag }}">{{ tag }}</button>
    {%- endfor %}
  </div>
  {%- endif %}

  {%- assign sorted_projects = site.projects | sort: "importance" -%}
  <div class="grid" id="projects-grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
</div>

<script>
(function () {
  var activeTags = [];
  document.querySelectorAll('.tag-filter-btn').forEach(function (btn) {
    btn.addEventListener('click', function () {
      var tag = this.dataset.tag;
      var idx = activeTags.indexOf(tag);
      if (idx === -1) { activeTags.push(tag); this.classList.add('active'); }
      else { activeTags.splice(idx, 1); this.classList.remove('active'); }
      document.querySelectorAll('#projects-grid .grid-item').forEach(function (item) {
        if (activeTags.length === 0) { item.style.display = ''; return; }
        var tags = (item.dataset.tags || '').split(',').map(function (t) { return t.trim(); });
        item.style.display = activeTags.some(function (t) { return tags.indexOf(t) !== -1; }) ? '' : 'none';
      });
    });
  });
})();
</script>
