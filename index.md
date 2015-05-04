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

<h2> IPython notebook </h2>
URLを`hoge.html`から`hoge.ipynb`に変えるとIPython Notebookのファイルに飛びます．

- [週刊少年ジャンプの順位推移の可視化](/ipynb/jump-vis.html)
- [将棋ウォーズで削除されたアカウントの棋譜を検討してみる](/ipynb/junpe_.html)
- [ニコニコ動画のコメントを可視化する](/ipynb/nico-an.html)
