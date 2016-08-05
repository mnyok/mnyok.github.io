---
layout: page
title: Tags 

---

{% comment %}
=======================
The purpose of this snippet is to list all the tags you have in your site.
=======================
{% endcomment %}
{% for tag in tags %}
  <a href="#{{ tag | slugify }}"> {{ tag }} </a>
{% endfor %}

<div class="page-content wc-container">
	<div class="post">
		<h1>Tags</h1>  
		<ul>
			{% for tag in site.tags %}
			<li><a href="{{site.baseurl}}/tag/{{ tag[0] }}">{{ tag[0] }}</a></li>
			{% endfor %}
		</ul>
	</div>
</div>
