---
layout: page
permalink: /publications/
title: Publications
description: International Conferences, Journals, and Workshops
nav: true
nav_order: 1
---
<!-- _pages/publications.md -->
<div class="publications">

<!--
<div markdown="0" class="container">
<div  class="col-sm-10">
  <input type="text" class="form-control" id="search-input" placeholder="Search...">
  <br>
  <div class="list-group mt-5" id="results-container">
  </div>
</div>
</div>
-->

{% bibliography -f {{ site.scholar.bibliography }} %}

</div>
