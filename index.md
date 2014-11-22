---
layout: page
title: メインページ
tagline: 工事中
---
{% include JB/setup %}

<h2> 最新5記事 </h2>

<ul class="post-list">
  {% for post in site.posts limit:5 %}
    <li>
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
		  <h4>
<a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
	</h4>
    </li>
	<br>
  {% endfor %}
</ul>
