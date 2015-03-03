---
layout: post
title: "ニコニコ動画のコメントクローラーを作った"
description: "自作クローラーでニコニコ動画のコメントを取得し，適当に可視化した"
category: data-an
tags: [Python, graph, ニコニコ動画]
---
{% include JB/setup %}


とある解析がしたかったので，自作クローラーでニコニコ動画のコメントを取得し，とりあえず適当に可視化した．


# コメントの収集方法

自作のクローラーを使って取得した．取得したクローラーのソースコードはGistにアップした．

- [NicoCrawler.py - Gist](https://gist.github.com/tosh1ki/10be02aa4599d5ff96ad)

大体2015/02/24-28にかけて取得した．取得する動画については[2014秋アニメ一覧のページ](http://ch.nicovideo.jp/2014fall_anime)に載っているアニメの過去放送分の動画のその時点での全てのコメントを取得した．収集したコメントは13,074,945件，ファイルサイズは2.2GBだった．

取得のログによると，なぜかダブってコメントを収集していたり取得漏れがあったりしたが，*気にならない程度なのでとりあえず話を進める*．Gistにアップしたソースコードではそういったバグは多分取り除かれていると思う．


# Bokehによる可視化

[Bokeh](http://bokeh.pydata.org/en/latest/index.html)(ボケェ)という，いかした名前のPythonのビジュアライゼーションのパッケージの存在を最近知ったので，収集したコメントデータでとりあえず試してみた．

面白い(?)例として，コメント中のプレミアム会員の割合を調べてみた．コメントには`premium`属性がついていて，そのコメントがプレミアム会員によるものかどうかがわかる．そこで，縦軸を**「プレミアム会員によるコメント(プレミアムコメント)の数」**，横軸を**「全てのコメント(プレミアム,非プレミアム)の数」**として，各動画をプロットしてみた．マウスオーバーで各点がどの動画に対応しているかが表示される．

<iframe src="/image/2015-03-01/comment_vs_pcomment.html" name="sample" width="620" height="480" scrolling="no"></iframe><br/>
[上のグラフ](/image/2015-03-01/comment_vs_pcomment.html)

緑色の直線は「コメント数=プレミアムコメント数」を表す．緑色の直線の上に乗っているアニメが割とあることがわかる．

その他に描いたグラフは以下のIPython Notebookに載っている．

- [nico-an.html](/ipynb/nico-an.html)


# その他
今回作ったクローラーを使えば，例えば以下のようなデータも取れる．

<blockquote class="twitter-tweet" lang="ja"><p>ニコニコ動画ごちうさ第1話の日毎のコメント数推移 <a href="http://t.co/FQQS0Fu2cH">pic.twitter.com/FQQS0Fu2cH</a></p>&mdash; tosh1ki (@tosh1ki) <a href="https://twitter.com/tosh1ki/status/572008364261031936">2015, 3月 1</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" lang="ja"><p>ピーク(&gt;6000)の理由はコメントの内容から推測するにそれぞれ左から，1話放送，12話放送，100万回再生達成?，150万回?，わからん，200万回?，キャラの誕生日，年越し?，2期発表x2，キャラの誕生日，かな？</p>&mdash; tosh1ki (@tosh1ki) <a href="https://twitter.com/tosh1ki/status/572015008483160064">2015, 3月 1</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


# 参考文献

- [niconicoのメッセージ(コメント)サーバーのタグや送り方の説明 - 神の味噌汁青海](http://blog.goo.ne.jp/hocomodashi/e/3ef374ad09e79ed5c50f3584b3712d61)
- [ニコニコ動画・ニコニコ生放送のコメント取得　備忘録 : まぢぽん製作所](http://blog.livedoor.jp/mgpn/archives/51886270.html)
