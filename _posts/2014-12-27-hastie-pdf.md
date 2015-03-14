---
layout: post
title: "Hastie本のPDFを使いやすくする"
description: "無料でダウンロードできるThe Elements of Statistical LearningのPDFが使いにくいので，ブックマークをつけて余白を切り取る．"
category: 備忘録
tags: [Python,備忘録]
---
{% include JB/setup %}

無料でダウンロードできる[The Elements of Statistical Learning](http://statweb.stanford.edu/~tibs/ElemStatLearn/)のPDFが使いにくいので，ブックマークをつけて余白を切り取る．

以下の方法を[Convex Optimization – Boyd and Vandenberghe](http://stanford.edu/~boyd/cvxbook/)にも試したところ，下図のように使いやすいPDFが得られた．

<img src="/image/2014-12-27/example1.png" alt="example1" style="width:50%;height:auto;"><img src="/image/2014-12-27/example2.png" alt="example2" style="width:50%;height:auto;">

PDF閲覧には[ezPDF Reader](https://play.google.com/store/apps/details?id=udk.android.reader&hl=ja)を使用した．

# やったこと

## 事前準備
[The Elements of Statistical Learning - Data Mining, Inference, and Prediction, Second Edition](http://www.springer.com/computer/ai/book/978-0-387-84857-0) の table of contents のPDFですべてのテキストを選択してTXTファイルにコピペしておく[^haihu]．このままだと目次のページのヘッダ(ex. `xiv Contents`)が紛れていたり，改行がおかしくなっていたりするので手作業で修正しておく．

参考文献などを参考にしてpdftkや[briss](http://sourceforge.net/projects/briss/)をインストールしておく．

[^haihu]: 配布されているHastie本の目次はタイトルとページ数がなぜか別カラムになっているので同じことをしてもうまく行かない

## 作業手順

pdftkを使ってHastie本のPDFからブックマーク情報を取り出す

	pdftk Hastie.pdf dump_data > bookmark.txt

自作のスクリプトでpdftkが読み込める形式のブックマーク情報を書き出す

	python pdfmine.py > mytoc.txt

`bookmark.txt` と `mytoc.txt` を結合する

	cat bookmark.txt mytoc.txt > new-bookmark.txt

pdftkを使って生成したブックマークをHastie本のPDFに合成する

	pdftk Hastie.pdf update_info new-bookmark.txt output temp.pdf

brissを起動

	java -jar ./path/to/briss-0.9.jar temp.pdf

参考文献を参考にして好きなように余白を切り取ったら，エクスポートする

完成

# まとめ
最高だった

# ソースコード
<script src="https://gist.github.com/tosh1ki/347d7b395c0104ec42ef.js"></script>

# 追記

PDF版で空白のページが削除されている場合[^empty]は自分で空白のページを挿入する必要がある．

[^empty]: 例えば，チャプターの境目などにある空白ページがなぜか削除されていたりした．

{% highlight bash %}
echo "" | ps2pdf -sPAPERSIZE=jisb5 - empty.pdf
pdftk A=before.pdf B=empty.pdf cat A1-98 B1 A99-119 B1 A120-158 B1 A159-end output inserted.pdf
{% endhighlight %}

ps2pdfなどで適当な空白ページを作成して，pdftkで挿入する．


# 参考文献
- [PDFのしおりを編集する \| CMSの構築ならRCMS \- あらゆる要望に応える最強のCMS](https://www.r-cms.jp/rcms-develop/list/article/id=323)
- [PDFをBrissでkindle用に余白を切り取る — そこはかとなく書くよん。](http://tdoc.info/blog/2013/01/11/briss_kindle.html)
- ["The Elements of Statistical Learning"のpdfの余白を削る - Wolfeyes Bioinformatics beta](http://yagays.github.io/blog/2013/05/22/esl-pdf-trimming/)
- [insert a blank page into a PDF using ghostscript or pdftk - Unix & Linux Stack Exchange](http://unix.stackexchange.com/questions/15992/insert-a-blank-page-into-a-pdf-using-ghostscript-or-pdftk)
- [pdftk - Insert a blank page between each existing page in a PDF document - Stack Overflow](http://stackoverflow.com/questions/12942486/insert-a-blank-page-between-each-existing-page-in-a-pdf-document)
