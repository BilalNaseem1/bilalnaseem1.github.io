---
layout: page
permalink: /research/
title: research
description: Peer-reviewed publications — update with your actual paper details.
years: [2025, 2024, 2023]
nav: false
nav_order: 4
---
<div style="text-align: right"> <small>* equal contribution</small> </div>
<!-- _pages/publications.md -->
<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
