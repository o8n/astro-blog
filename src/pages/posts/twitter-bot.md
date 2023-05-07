---
title: "つんくさんのBotを作る"
pubDate: 2020-05-31
draft: false
share: true
tags: ["Ruby", "Python", "Heroku", "hobby"]
layout: ../../layouts/MarkdownPostLayout.astro
---

唐突ですが、つんくさんのツイートがスキちゃんです。例えば

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ワンちゃんならシッポグルグルやけどね。 <a href="https://t.co/Jx55aKIXm1">https://t.co/Jx55aKIXm1</a></p>&mdash; つんく♂ (@tsunkuboy) <a href="https://twitter.com/tsunkuboy/status/1264949286724341765?ref_src=twsrc%5Etfw">May 25, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


つんくさんが作詞作曲したハロプロ研修生の「テーブル席空いててもカウンター席」という曲に**ワンちゃんならおしっぽグルグルだよ**という歌詞があります。曲を聞いても、この歌詞は全然意味が分かりません。これが堪らないのです。

<!--more-->

モーニング娘を始め、つんくさんが作詞作曲を手掛けたハロプロ楽曲は名曲も珍曲も含めて、彼のワードセンスが我々を「？？？」という気持ちにさせてくれてすごく楽しい気持ちになります。深掘りしすぎるとオタクしかついてこれないのでこのポストでは多く触れません。


あとつんくさんはTwitterを2010年からやっていて、その頃使われていた（けど今はあまり聞かない）インターネット用語を2020年になっても使います。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">うす！<br>ただ、つべるんでなく、Gメってくださいとのことでした！<br>なので、Twitterとは別に待ってます！ <a href="https://t.co/MeC2x1i3Bs">https://t.co/MeC2x1i3Bs</a></p>&mdash; つんく♂ (@tsunkuboy) <a href="https://twitter.com/tsunkuboy/status/1207180221536985088?ref_src=twsrc%5Etfw">December 18, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

2019~20年現在YouTubeを「つべ」って言ってる人、しかも動詞として使っている人をつんくさん以外に知りません。「うp」とかもよく仰ります。そして、、**Gメる**というキラーワード。お初にお目にかかりました。堪りませんな。

つんく（本名：寺田光男）さんはなんと言っても、日本人なら誰でも歌える「LOVEマシーン」や「恋愛レボリューション21」を産み出した偉大な音楽家ですから、それなりの独特の感性や言語感覚をお持ちなんだな〜といつも見ています。

そんなわけで、Twitterでのつんくさんの独特の言い回しや歌詞解説などをシャブり続けたくなる衝動に駆られたので、チャッとTwitter Botを作りました。

# 手順

1. tweepyで過去のツイートを取得しcsvダウンロード
1. Rubyスクリプトを記述
1. できたアプリケーションをHerokuにデプロイし自動投稿の設定

今までTwitterのAPIを扱うことはあったのですが、Botは作ったことがなくてデプロイに少し手こずりました。とはいえ簡単にできました。

### 参考にしたページ

- Tweepyでツイート取得 [https://qiita.com/i_am_miko/items/a2e5168e619ed37afeb9](https://qiita.com/i_am_miko/items/a2e5168e619ed37afeb9)
  - なんとなくPythonを使ってみたくなったのです
  - ツイート数の指定をしたわけではないですが、後々みたら2014年までのツイートしか取得できてませんでした。なんでだろう

- Rubyスクリプト
  - APIキーを扱うtwitterライブラリを使用
  - ツイートは予めコード内の配列に持たせておきます。おかげで行数は数百に及びます。もう少しスマートにできたら…~~12,スマートに…~~

- Herokuの設定[https://trap.jp/post/458/](https://trap.jp/post/458/)
  - ソースコードのenvファイルだけでなくHeroku側でも環境変数を設定するのが味噌です。

# できたもの

[https://twitter.com/bot_tsunku](https://twitter.com/bot_tsunku)

![.](/static/images/post/tsunku.png)

今のところ1時間に一回呟くようにしています。うるさいかもしれない。Berryz工房のデビューシングル「あなたなしでは生きてゆけない」のコーラス並にうるさいかも。じゃあええか!?

あと、つんくさんは毎月1日に「おついたち」ブログをお書きになるので、bot側でも毎月1日はおついたち関連のツイートを流すとか、天気に関するツイートもよくされるので、雨が降っていたらそれ関連のツイートを流すなど、そういうディテールに拘ってボチボチ運用できたらいいなと夢見ております。マルコフ連鎖を駆使して「つんくさんが言いそうなことを言う」とか。今のところは完全にランダムに文字列を返しているだけです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ま、しかし、それも青春か。笑</p>&mdash; つんく♂ (@tsunkuboy) <a href="https://twitter.com/tsunkuboy/status/1026253371349643264?ref_src=twsrc%5Etfw">August 5, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
