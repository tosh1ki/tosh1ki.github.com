---
layout: post
title: "北海道大学の成績分布"
description: ""
category: data-an
tags: [data-an,Python,graph]
---
{% include JB/setup %}

[北海道大学 成績分布ＷＥＢ公開システム ](http://educate.academic.hokudai.ac.jp/seiseki/GradeDistSerch.aspx)なるものがある．

>「成績評価の公平性を確保し，学生及び第三者に対する説明責任を果たす」という本学の方針に則り，全学教育科目及び専門科目について，
本学ＨＰ上に成績分布・ＧＰ（grade point）の平均値の公表を進めています。

とのことで，大体の講義の大まかな成績の分布を公表しているということらしい．

面白そうなので少し調べてみる．

## データが怪しい？ ##
何箇所かおかしい部分があったので，一応書いておく．

### 履修者数がおかしい？ ###
>【公開は履修者が５名以上の科目を対象としています。】

（[北海道大学 成績分布ＷＥＢ公開システム ](http://educate.academic.hokudai.ac.jp/seiseki/GradeDistSerch.aspx)，2014/04/05 17時頃参照）

とのことだが，試しに「2013年度 １学期 学士課程 工学部 授業科目・担当教員別」を指定して出力してみると

![](/image/2014-10-03/20140405170345.png)

（担当教員名の右は「履修者数」の欄）

履修者数1の科目がある．

分析上問題は無いので，とりあえず気にしないでおこう．

### htmlがおかしい？ ###
「2013年度 １学期 学士課程 医学部 授業科目・担当教員別」の出力結果

{% highlight html %}
<td align="Right">
  <font color="0">0.0
  </font>
</td>
<td align="Right">
  <font color="0">2.43font>
</td>
</tr>
<tr>
{% endhighlight %}

`2.43font>`となっていて非常に辛い．これが原因のエラーに対処するのにかなり時間を使った．

正しくは`2.43</font>`とするべきだろう．ちなみにChromeでは正常に表示されてしまっていた．

## 箱ひげ図を描く ##
「2013年度 １学期 学士課程 授業科目・担当教員別」の条件で成績を出力して保存する，という作業をすべての学部について行う．

まず「2013年度 １学期 学士課程」の各学部の講義数を調べる．

|----|------|
|学部|講義数|
|:--|:--|
|文学部|172|
|教育学部|68|
|法学部|46|
|理学部|160|
|医学部|175|
|歯学部|26|
|薬学部|91|
|工学部|403|
|農学部|136|
|獣医学部|47|
|水産学部|57|
{: .table .table-striped}

次に，出力したデータをもとに箱ひげ図を描く．

以下の箱ひげ図は，各学部各講義の 不可の割合，秀の割合，GPAの平均 を学部ごとにまとめて箱ひげ図にしたものである．ここで，「不可」は評価点59点以下，「秀」は評価点90点以上を表す[^gpa]．データが「各講義の秀の割合(%)」といった形式でしか与えられていないので，箱ひげ図から情報を読み取るのが少し難しいが仕方ない．

[^gpa]: cf. [「秀」評価及びＧＰＡ制度の実施について（報告）](http://educate.academic.hokudai.ac.jp/GPA/gpa.pdf)

![](/image/2014-10-03/boxplot-huka.png)

不可の箱ひげ図を見ると，講義の多さを考慮に入れても理・工学部の外れ値が目立っている．特に，理・工学部では半分以上の学生が不可を付けられる科目が結構数あるのがすごい．また反対に，医・歯・薬・獣医学部あたりは学部全体の傾向として不可が少ない．

この辺の傾向は，「学部による全体の科目必修・選択科目の割合の違い」で説明がつきそうな気がする．

![](/image/2014-10-03/boxplot-shu.png)

秀の箱ひげ図からは，すべての学部でそれほど目立った違いはないという印象を受ける．しきい値をどの辺に設定するかにもよるが，文・歯・工学部あたりは特に秀が多いと言える．

こういった評価の割合などには，

 - 学部による全体の科目必修・選択科目の割合の違い
 - 講義の受講者数の分布

などが影響してくると予想できる．よって不可が多いとか秀が少ないとかで学部の厳しさは測れない気もする．

![](/image/2014-10-03/boxplot-gpa.png)

GPAの箱ひげ図も描いたが，上と同じ理由でGPAの箱ひげ図の評価も難しい．

## ソースコード ##
すべての学部について出力結果を保存して，それらすべての文字コードをunicodeに直してから，以下のスクリプトを使う．


{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import BeautifulSoup as bs
import pandas as pd
import matplotlib.pyplot as plt

if __name__ == '__main__':
    gakubu_list = [
        u'文学部',u'教育学部',u'法学部',u'理学部',u'医学部',
        u'歯学部',u'薬学部',u'工学部',u'農学部',u'獣医学部',
        u'水産学部']

    result_list = []
    colu = u'GPA'

    for gakubu in gakubu_list:
        filename = './txt/' + gakubu + '-u.txt'
        htmltext = open(filename).read()

        print filename

        soup = bs.BeautifulStoneSoup( htmltext, 
            convertEntities=bs.BeautifulStoneSoup.HTML_ENTITIES)
        table = soup.find('table',attrs={'id':'rdlGrid_gridList'})

        data = []
        for i,tr in enumerate(table.findAll('tr')[0:-1]):
            rowlist = []

            for j,td in enumerate(tr.findAll('td')):
                ## 0列目と0~4行目は数字でない
                if j <= 4 or i == 0:
                    rowlist.append( td.find(text=True) )
                elif j > 4:
                    rowlist.append( float(td.find(text=True)) )
            data.append(rowlist)

        frame = pd.DataFrame(data[1:-1],columns=data[0])
        
        result_list.append( frame[colu] )

    ## x軸の各箱ひげ図のラベルをつけるためによくわからない方法を使う
    ax = subplot(111)
    ax.boxplot(result_list)
    ax.set_xticklabels(gakubu_list,rotation=45)
    ax.set_ylim(-10,110) ## 手で調整
    ax.set_title(colu)
{% endhighlight %}

## 参考にした ##
[Amazon.co.jp： Pythonによるデータ分析入門 ―NumPy、pandasを使ったデータ処理: Wes McKinney, 小林 儀匡, 鈴木 宏尚, 瀬戸山 雅人, 滝口 開資, 野上 大介: 本](http://www.amazon.co.jp/Python%E3%81%AB%E3%82%88%E3%82%8B%E3%83%87%E3%83%BC%E3%82%BF%E5%88%86%E6%9E%90%E5%85%A5%E9%96%80-%E2%80%95NumPy%E3%80%81pandas%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%87%E3%83%BC%E3%82%BF%E5%87%A6%E7%90%86-Wes-McKinney/dp/4873116554)
