---
layout: post
title: "GASでGoogle Contacts上の電話番号を修正する"
description: ""
category: 備忘録
tags: [Google Apps Script,JavaScript]
---
{% include JB/setup %}

なぜか Google Contacts 上のほとんどの電話番号が`090...`でなく`90...`のように，頭の`0`が取れた状態になっていた．そこで Google Apps Script で電話番号の使える形式に直す．

電話番号のfieldに入っている文字列の頭が`0`でない場合は頭に`0`を付けた電話番号を追加して，元の電話番号のデータは削除する．自分の状況でそれなりにうまくいくような実装でしかない．

<script src="https://gist.github.com/tosh1ki/6c4a1c8b64c11dd7450a.js"></script>
