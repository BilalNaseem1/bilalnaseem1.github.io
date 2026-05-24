---
layout: page
title: Portfolio
permalink: /projects/
description: A collection of AI, data engineering and machine learning projects — pipelines, platforms, and production systems.
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
    <button class="tag-filter-btn active" data-tag="__all__">All</button>
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
  var active = [];
  document.querySelectorAll('.tag-filter-btn').forEach(function (btn) {
    btn.addEventListener('click', function () {
      var tag = btn.dataset.tag;
      if (tag === '__all__') {
        active = [];
        document.querySelectorAll('.tag-filter-btn').forEach(function (b) { b.classList.remove('active'); });
        btn.classList.add('active');
      } else {
        document.querySelector('.tag-filter-btn[data-tag="__all__"]').classList.remove('active');
        if (btn.classList.contains('active')) {
          btn.classList.remove('active');
          active = active.filter(function (t) { return t !== tag; });
        } else {
          btn.classList.add('active');
          active.push(tag);
        }
        if (active.length === 0) {
          document.querySelector('.tag-filter-btn[data-tag="__all__"]').classList.add('active');
        }
      }
      document.querySelectorAll('#projects-grid .grid-item').forEach(function (item) {
        if (!active.length) { item.style.display = ''; return; }
        var tags = (item.dataset.tags || '').split(',').map(function (t) { return t.trim(); });
        item.style.display = active.some(function (t) { return tags.indexOf(t) !== -1; }) ? '' : 'none';
      });
    });
  });
})();
</script>
