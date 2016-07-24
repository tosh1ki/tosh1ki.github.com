---
layout: post
title: "word2vec のソースコードを解読する"
description: "読みにくいことで有名なword2vecのソースコードを解読する"
category: ソースコードを読む
tags: [ソースコードを読む, NLP]
---
{% include JB/setup %}


1,2年くらい前から自然言語処理界隈などを賑わせているらしい[word2vec](https://code.google.com/p/word2vec/)[^mikolov2013]は，その実行速度が**非常に速い**ことで有名でもある[^fast][^fast2]が，その一方で，ソースコードが非常に読みにくいことが一部で知られている[^unreadable]．例えば，`a`, `b`などの1文字の変数があったりとか，一部で論文に一切記載のない手法の改良?が数点施されていたりする．本記事では，諸事情でword2vecの解読をしたけど結局その話は使わないことになった筆者が，**「高速化の技法」「論文に記載のない変更点」「その他」**の3点について備忘録として雑に説明していく．

[^mikolov2013]: Mikolov, T., Chen, K., Corrado, G., & Dean, J. (2013). Distributed Representations of Words and Phrases and their Compositionality. Nips, 1–9. Retrieved from <http://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf>
[^fast]: <https://twitter.com/unnonouno/status/625677637864689667>
[^fast2]: <https://twitter.com/himkt/status/530941194348404737>
[^unreadable]: <https://twitter.com/unnonouno/status/558908136229060608>

なお，最適化や更新式の導出などの話はここではしないので，その辺については適当な解説論文[^goldberg2014][^rong2014]を参考にしてほしい．

[^goldberg2014]: Goldberg, Y., & Levy, O. (2014). word2vec Explained: Deriving Mikolov et al.’s Negative-Sampling Word-Embedding Method. arXiv Preprint arXiv:1402.3722, (2), 1–5. Retrieved from <http://arxiv.org/abs/1402.3722>
[^rong2014]: Rong, X. (2014). word2vec Parameter Learning Explained. CoRR, abs/1411.2, 1–19. Retrieved from <http://arxiv.org/abs/1411.2738>

また，自分でも解読をしたいという稀有な人には，`gensim.models.word2vec` の Python 実装のソースコード[^gensim]も並列して読むことをおすすめする．

[^gensim]: <https://github.com/piskvorky/gensim/blob/develop/gensim/models/word2vec.py> の`class Word2Vec` の部分

この記事では，"word2vec" はMikolovらの手法を実装したC言語のプログラム (<https://code.google.com/p/word2vec/>) を指す言葉として用いる．


# 高速化の技法

word2vec では，

* exp 関数の計算の簡略化
* 乱数の関数呼び出しコストの削減

の2つの技法を用いて，高速化を図っている．以下ではそれぞれについて見ていく．

## exp 関数の計算の簡略化

ロジスティックシグモイド関数の計算をする部分で必要な exp 関数の計算が重いということで，ルックアップテーブル(`expTable`)を構成し，単語ベクトルの最適化の段階では，このテーブルの値を使ってロジスティックシグモイド関数を計算している．正弦関数などの比較的重い関数の計算を何回も行う必要がある場合，関数の部分をルックアップテーブルで置き換える，というのはよくある高速化のための手段らしい．

これについては word2vec-toolkit の Google Groups において，Mikolovが以下のように解説している[^ggroups]．

> As you can notice, the usual exp() would be quite expensive to call, and thus there is a look-up table based approximation which is computed over a certain interval.

[^ggroups]: <https://groups.google.com/d/msg/word2vec-toolkit/RBENshFC7Gc/xt5MnyGT_PgJ>

## 乱数の関数呼び出しコストの削減

word2vec では negative sampling などで乱数を用いる部分がある．このとき，word2vec では以下のように，C言語の `rand` 関数を使わずに，わざわざ自前で線形合同法の乱数生成器をベタ書きしている．これはおそらく関数呼び出しコストを削減するための処置?だろう．とはいえ，インライン関数とかマクロとかで書けばわざわざ何回も乱数生成器をベタ書きせずに済むのでは…? という気もする．

{% highlight C %}
next_random = next_random * (unsigned long long)25214903917 + 11;
if (ran < (next_random & 0xFFFF) / (real)65536) continue;
{% endhighlight %}
<https://code.google.com/p/word2vec/source/browse/trunk/word2vec.c#397>

## Negative sampling 用の Unigram Table の構成

`InitUnigramTable` 関数で Unigram table `table` (tableという名前のグローバル変数) を構成している．一様ランダムに `table` の要素を取得することで，原論文[^mikolov2013]の 2.2 の最後の方に書かれている， negative sampling に必要な $P_n(w)$ が実現できる．

# 論文に記述のない変更点
論文と実装で異なっている点としては，発見しただけでも以下の2点が挙げられる．

* 並列化
* <s>contextの窓の幅をランダムで短くする</s> (Efficient Estimation of Word Representations in Vector Space に記載があった．)

以下それぞれについて見ていく．

**(追記)** 論文に記述のない変更点については，[岡崎先生の解説動画](https://youtu.be/EyL_TC17MkQ?t=25m20s)においてもいくつか指摘されている．


## 並列化
word2vec では，多少無理な並列化を行っている．これについては，専門家が書いた詳しい資料[^okazaki]があるのでそちらを参照して欲しい．きちんと計測したわけではないが，並列化により数倍は速くなるので，これも word2vec の速さの秘訣と言えるだろう．

[^okazaki]: Word2vecの並列実行時の学習速度の改善 [http://www.slideshare.net/naoakiokazaki/word2vec](http://www.slideshare.net/naoakiokazaki/word2vec)

## contextの窓の幅をランダムで短くする
word2vec の実装において，学習に使う単語を変える度に `b = next_random % window;` を使って窓の幅をランダムで短くしている．<s>これについては Google Groups での言及も見つけられなかったので，本当によくわからない．そういうものなんだろうか．

**(追記)** よく読んだら "Efficient Estimation of Word Representations in Vector Space" に記述があったが，特に理由は書いていなかった．


# その他

## sen の定義
プログラム実行時に `-train` で指定した訓練データは，`long long` 型の数字に変換された後，配列 `sen` に格納される．`sen` の要素は `vocab` におけるインデックスを表す．つまり，訓練データの `n` 番目の単語 (`sen[n]`) に対応する単語(を表す文字列)は，`vocab[sen[n]].word` で得られる．

## syn0, syn1neg の定義
`syn0`, `syn1neg` は，それぞれ単語, Negative sample のベクトルを1行に並べた配列である．

例えば，`sen[n]` に対応する単語の単語ベクトルに対応する部分は，`syn0[sen[n] * layer1_size]`,...,`syn0[sen[n] * layer1_size + (layer1_size - 1)]` で得られる．ここで，`layer1_size` は単語ベクトルの次元数である．


# 感想
途中で紹介した word2vec の Google groups には，週に数個ほど質問が飛んでいたりするのだが，この記事で紹介したようなことがらについての質問もたまに見受けられる．わかりにくいコードを書くと後で大変になるんだな，ということを学んだ．また，非常に短い変数名を使うと，例えば「この変数 `b` ってなんだ？」というような疑問を抱いた時に非常に検索しにくくなる，といった問題が発生することを学んだ．

<br />
<br />
<br />
<br />
<br />

(ここから word2vec ポエム)

「"king" - "man" + "woman" に最も近い単語ベクトルが "queen" になるんですよ」みたいな説明をよく見るが，大抵は "king" か "woman" が最も近くに来ることが多いと気がする．後処理でクエリに入っている単語 (king, man, woman) を弾いていることに触れないのは，ずるいのではという感じがする．

「word2vecはプログラムの名前であって，手法の名前ではない」という意見があるが，実際に手法としても今回紹介したように割と大きな変更点があるのだから，本来は一緒くたにするべきではないと思う．とはいえ自分は面倒なので使い分けることはしないが．

(ポエムここまで)


<br />
<br />
<br />
<br />
<br />
