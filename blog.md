---
layout: page
title: Blog
permalink: /blog/
---

#### Blog

<div class="blog-posts">
	<ul>
	  {% for post in site.posts %}
	    <li>
	      <b><a class="post-title" href="{{ post.url }}">{{ post.title }}</a></b>
	      <p class="post-date">{{ post.date | date_to_long_string}}</p>
	    </li>
	  {% endfor %}
	</ul>
</div>
