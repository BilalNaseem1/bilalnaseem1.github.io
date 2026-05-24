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
  <div class="tag-dropdown">
    <button class="tag-dropdown-toggle" id="tag-dropdown-btn">
      All projects <span class="caret">▾</span>
    </button>
    <div class="tag-dropdown-menu" id="tag-dropdown-menu">
      {%- for tag in all_tags %}
      <label class="tag-dropdown-item">
        <input type="checkbox" value="{{ tag }}"> {{ tag }}
      </label>
      {%- endfor %}
    </div>
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
  var btn  = document.getElementById('tag-dropdown-btn');
  var menu = document.getElementById('tag-dropdown-menu');
  if (!btn || !menu) return;

  btn.addEventListener('click', function (e) {
    e.stopPropagation();
    menu.classList.toggle('open');
    btn.classList.toggle('open');
  });

  document.addEventListener('click', function (e) {
    if (!e.target.closest('.tag-dropdown')) {
      menu.classList.remove('open');
      btn.classList.remove('open');
    }
  });

  menu.querySelectorAll('input[type="checkbox"]').forEach(function (cb) {
    cb.addEventListener('change', function () {
      var active = Array.from(menu.querySelectorAll('input:checked')).map(function (i) { return i.value; });
      btn.innerHTML = (active.length ? active.join(', ') : 'All projects') + ' <span class="caret">▾</span>';
      document.querySelectorAll('#projects-grid .grid-item').forEach(function (item) {
        if (!active.length) { item.style.display = ''; return; }
        var tags = (item.dataset.tags || '').split(',').map(function (t) { return t.trim(); });
        item.style.display = active.some(function (t) { return tags.indexOf(t) !== -1; }) ? '' : 'none';
      });
    });
  });
})();
</script>
