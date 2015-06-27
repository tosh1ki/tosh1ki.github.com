---
layout: post
title: "「王手飛車をかけた方が負ける」は本当か"
description: "将棋ニコ生などで頻繁に聞く「プロの試合では王手飛車をかけたほうが負ける」というジンクス(格言?)を確かめる"
category: data-an
tags: [data-an, 将棋, Python]
---
{% include JB/setup %}

将棋ニコ生などで頻繁に聞く[^ref]**「プロの対局では王手飛車をかけたほうが負ける」**というジンクス(格言?)が本当なのかを自作の将棋解析ライブラリを使って確かめる．

[^ref]: 要出典

## 「プロの試合では王手飛車をかけたほうが負ける」とは

Wikipediaの「[両取り](https://ja.wikipedia.org/wiki/%E4%B8%A1%E5%8F%96%E3%82%8A)」の項目には，以下のように書いてある．

> 棋士同士の対局の場合は、「王手飛車をかけた方が負ける」という言葉もある（棋士は王手飛車になる可能性まで踏まえて指しているため）。

また，マイナビの本[^mynavi]には次のように書いてある．

> 「プロは王手飛車をかけたほうが負ける」というセオリー(?)があります。プロが王手飛車を見落とすはずはないので、飛車を取られてもいい読みの組み立てをしているから、という意味です。

[^mynavi]: 週刊将棋編集部「ご存じですか?~将棋用語のおもしろ辞典」,マイナビ (2012)

また，実際に確かめてみた例として，[知恵袋のとある質問](http://detail.chiebukuro.yahoo.co.jp/qa/question_detail/q1395585673)の回答中に「『近代将棋』 2002年9月号に，王手飛車をかけたほうが勝率が良かったと書いてあった」という旨の情報が挙げられている．

で，実際どうなのか気になったので確かめてみることにする．今回は，「プロの対局において，王手飛車をかけた方の勝率が低い」を仮説として，検証を進めていくことにする．


## 「王手飛車」の定義
まず始めに「王手飛車」の定義をきちんとしておく必要がある．

「王手飛車」は単に「王手飛車取り」を指す用語である．では言葉通りにある駒が相手の王と飛車に効いている局面だけをカウントすればよいのかというと，そうではない．この理由を見るために，以下の局面図を見て欲しい．

![単に両取り](/image/2015-06-24/forking0.png)

これは「羽生善治 vs. 谷川浩司 (王位戦, 2003/09/08)」の77手目☗４二角成の局面である．この局面では，先手の指した手によって馬の効きが王と飛車にあたっているが，後手は次に☖同玉と取るため，王も飛車も取っていないという状況になる．単に「ある駒が相手の王手と飛車に効いている局面」を検索すると，こういった局面が大量にヒットしてしまう．

ということで今回の疑問を解決するためにあたり，**「王手飛車」をどう定義して抽出するかが割と難しい問題である**ことがわかる．この問題を回避するために，色々な「王手飛車」の定義が考えられる[^iroiro]．今回は，どちらかの棋士が以下の両方の条件を満たす手を指したとき，その棋士が「王手飛車」をした，と定義することにする．

* $n$ 手目に指した手によって，相手の王と飛車(or 龍)にある駒が同時に効いている局面になる
* 上が満たされている状況で，$n + 2$ 手目に飛車(or 龍)を取る

この定義だと，例えば「終盤のごちゃごちゃした局面で駒を取り合っている」みたいな局面もヒットしてしまいそうな気がする．しかし，実際に実験結果を見てみるとそういった局面はあまり抽出されていなかったため，この定義で話を進めることにする[^please]．

[^iroiro]: 「王飛両取りしている駒に相手の駒が効いてない」とか
[^please]: 何か他にいい「王手飛車」の定義があったら教えてください


## 実際に確かめる

### 将棋解析専用ライブラリ pyogi

王手飛車を検出する，となると駒の効きをきちんと取り扱う必要がでてくるため，アドホックに書いたプログラムで簡単に確かめる，ということが厳しいと予想できる．そこで，将棋解析専用のライブラリをPythonでざっくり作った．

* [tosh1ki/pyogi - GitHub](https://github.com/tosh1ki/pyogi)

この記事に貼ってある局面図もこのライブラリを使って描画している．更にKI2->CSAの変換など色々な機能を鋭意実装中．**issue, プルリク大歓迎**

今回は，`pyogi` のサンプルコードの中にある `search_forking_pro.py` を使って集計を行う．集計の詳細については GitHub 内のコードを見て欲しい．

### 使用したデータ
[2chkifu.zip](https://code.google.com/p/zipkifubrowser/downloads/detail?name=2chkifu.zip&can=2&q=)を使う．

### 結果

#### どんな局面が「王手飛車」として抽出される?
例えば以下のような局面がヒットする．

##### 島朗 vs. 三浦弘行 (順位戦, 2003/09/05)
38手目: ３五角

![島vs三浦](/image/2015-06-24/forking1.png)

ちなみに後手勝ち

##### 鈴木大介 vs. 谷川浩司 (日本シリーズ, 2003/08/03)
155手目: ９二角

![鈴木vs谷川](/image/2015-06-24/forking2.png)

ちなみに先手勝ち

#### 集計した結果
2chkifu.zipの中から抽出できた41867件の棋譜をもとに，王手飛車が勝率に影響を与えるかを計算する．まず，各々の対局(棋譜)について「王手飛車の局面が発生したか?」と「王手飛車した方が勝ったか?」をそれぞれ判定する．それを集計した結果が以下の表である．

<table>
  <tr>
    <th></th> 
    <th></th>
    <th colspan="2"> 王手飛車した方が勝ったか? </th>
  </tr>
  <tr>
    <th></th>
    <td></td>
    <td>False</td>
    <td>True</td>
  </tr>
  <tr>
    <th rowspan="2"> 王手飛車の局面が発生したか? </th>
    <td>False</td>
    <td>39702</td>
    <td>0</td>
  </tr>
  <tr>
    <td>True</td>
    <td>777</td>
    <td>1388</td>
  </tr>
</table>

手始めに，王手飛車が発生する割合は $\frac{777 + 1388}{41867} \simeq 0.05$ より約5%であることがわかる．[先の知恵袋の回答](http://detail.chiebukuro.yahoo.co.jp/qa/question_detail/q1395585673)によれば，近代将棋による集計でも出現率が約5%だったということで，そんなに変な値が出ているというわけではなさそうだ．

そして，本題の「王手飛車した方が負ける」に関してだが，計算するまでもなく王手飛車をかけた方の勝率が良いことがわかる．具体的には，王手飛車をかけた場合，かけた方の勝率は $\frac{1388}{1388 + 777} \simeq 0.64$ であった．


## まとめ
* 将棋解析専用ライブラリを作った
* 今回のデータは仮説「プロの対局では王手飛車をかけたほうが負ける」を支持しない
* プロの言葉とはいえ鵜呑みにするのは良くない．権威にはデータで対抗していこう