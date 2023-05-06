---
title: "ActionMailerでSNSシェアボタンを作る"
pubDate: 2020-09-04T02:13:41+09:00
draft: false
share: true
images: ["static/images/post/blog.png"]
tags: ['Rails']
layout: ../../layouts/MarkdownPostLayout.astro
---
ActionMailerでSNSシェアボタンを作ろうとしたら少し苦戦しました。

Railsアプリケーションでメールを送信する場面があり、そこにSNSでシェアできる機能を作る機会がありました。

作る前は「TwitterとかFacebookが公式でバッジ提供してくれてるだろうからそれ使えば良いんじゃないの」と思っていました。

<!--more-->

facebook：[https://developers.facebook.com/docs/plugins/share-button?locale=ja_JP](https://developers.facebook.com/docs/plugins/share-button?locale=ja_JP)

twitter：[https://developer.twitter.com/en/docs/twitter-for-websites/tweet-button/overview](https://developer.twitter.com/en/docs/twitter-for-websites/tweet-button/overview)

公式バッジを使う場合、URLを静的に埋め込むことしかできません。例えば**動的にidだけを入れ替えることができません**。

ActionMailerだと、HTMLと同じ感じでシェアボタンをメール内に埋め込むということが難しいと分かりました。

結果として、**Twitter/Facebookシェアボタンの画像をerbファイルに貼り、その画像のhrefにシェアURLを与える**ことにしました。

ここで考えることは主に以下の4つ。

- ActionMailer内で画像を扱えるように設定する
- 画像にURLを貼り付ける
- SNSのurlのオプションの付け方
- `<a>`タグを用いるか`link_to`を用いるか

## ActionMailer内で画像を扱う

コントローラの場合と異なり、メイラーのインスタンスには受け取ったリクエストのコンテキストが一切含まれません。

このため、`:asset_host`パラメータを自分で指定する必要があります。

`:asset_host`がアプリケーション全体で一貫しているのと同様、`config/application.rb`でグローバルな設定を行えます。

勿論、`config/development.rb`など環境ごとに設定することもできます[^1]。

```rb
config.action_mailer.asset_host = 'http://example.com'
```

## 画像にURLを貼り付ける

テンプレート内で`<img>`タグを生成するために`image_tag`メソッドが使えます。

ActionView::Helpers::AssetTagHelperで定義されています。オプションの中で画像のサイズを指定したりできます。

```rb
= image_tag(source, options = {})
```

[https://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html#method-i-image_tag](https://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html#method-i-image_tag)

画像にURLを貼るために`link_to`メソッドを使います。ActionView::Helpers::UrlHelperで定義されています。

```rb
= link_to(name = nil, options = nil, html_options = nil, &block)
```

[https://api.rubyonrails.org/v5.2.3/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to](https://api.rubyonrails.org/v5.2.3/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to)

nameを指定する場所に`image_tag`メソッドを記述することで画像に対してリンクを設定することができます。

例

```rb
<%= link_to image_tag('btn.png'), '/books/index' %>

# 出力結果 -> <a href="/books/index"><img alt="Btn" src="/assets/btn.png" /></a>
```

## URLへのオプションの付け方

Twitter、Facebookのシェア用URLに付けられるオプションについてです。

Twitterでは

`https://twitter.com/intent/tweet`

というURLがシェア用URLとして存在します。使用できるパラメータは以下です。

- `text`: ツイート本文（UTF-8およびURLエンコードされている）
- `url`: リンク（URLエンコードされている）
- `hashtags`: ハッシュタグ
- `via`: アカウント名
- `related`: アカウント名
- `in-reply-to`: リプライ先の tweet_id

Facebookでもシェア用URLが用意されています。

```html
https://www.facebook.com/share.php?u=<url>
https://www.facebook.com/sharer/sharer.php?u=<url>
```

Facebookの方はurlパラメータがなく、twitterで用意されているtextなどのパラメータは使用できません。

## `<a>`タグを用いるか`link_to`を用いるか、そしてRails特有のこと

以下は失敗例です。ちなみにですが、例の中では「CDアルバムを扱うECサイト」を想定しています。アルバムを購入した時に届く購入完了メールにシェアボタンがついていて「購入しました」とSNSで主張できるというものです。

```html
<a href="https://twitter.com/intent/tweet?text=アルバムを購入しました&url=https://example.com/artists/#{@album.artist.account_name}/artworks/#{@album.artwork.id}&hashtags=example&via=example">
```

Railsを用いている場合、

`&url=https://example.com/artists/#{@album.artist.account_name}/artworks/#{@album.artwork.id}&hashtags=example&via=example`

という部分は`*_url`もしくは`*_path`を用いて表現する方法を考えるのが良いです。

この例だと

```txt
artist_artwork_url
もしくは
artist_artwork_path
```

になります。

`*_path`は相対パス、`*_url`は絶対パスです。rootを例にとると以下のようになります。

```rb
root_path -> '/'
root_url  -> 'http://www.example.com/'
```

`*_path`ヘルパーは、メール内では一切利用できません。

メールでURLが必要な場合は`*_url`ヘルパー、つまり絶対パスを使う必要があります[^2]。

上の場合`artist_artwork_url`を使います。

（viewなどでは`*path`を用い、リダイレクトする時のみ`*url`を用いるのが一般的です[^3]）

ここで、先ほどあげたシェアURL部分、

```html
https://twitter.com/intent/tweet
https://www.facebook.com/share.php?u=
```

などは`route.rb`でスコープにして共通化してあげましょう。

`config/route.rb`

```rb
scope host: "www.twitter.com", protocol: :https, port: nil do
  get '/intent/tweet', to: redirect("https://twitter.com/intent/tweet"), as: :twitter_share
end

scope host: "www.facebook.com", protocol: :https, port: nil do
  get '/sharer/sharer.php', to: redirect("https://www.facebook.com/sharer/sharer.php"), as: :facebook_share
end
```

こうすることで逐一URLを書かずにすみます。

```ruby
<%= link_to image_tag('twitter.jpg'), twitter_share_url(text: 'アルバムを購入しました', url: artist_artwork_url(@artwork.artist, @artwork), hashtags: 'example', via: 'example') %>
```

```ruby
<%= link_to image_tag('facebook.jpg'), facebook_share_url(u: artist_artwork_url(@album.artist, @album.artwork)) %>
```

## [2020.09.17追記] ActionMailerでのファイルの添付

上記を実行し、ローカル上では画像が表示されるものの、サーバーにあげると表示されませんでした。

Railsガイドにファイルの添付について説明がありました。[^4]

>Action Mailer 3.0はファイルをインライン添付できます。この機能は3.0より前に行われた多数のハックを基に、理想に近づけるべくシンプルな実装にしたものです。インライン添付を利用することをMailに指示するには、Mailer内のattachmentsメソッドに対して#inlineを呼び出すだけで済みます。

in `mailers/hoge.rb`

```ruby
attachments.inline['filename.jpg'] = File.read('/path/to/filename.jpg')
```

これをviewに反映させます。

>続いて、ビューでattachmentsをハッシュとして参照し、表示したい添付ファイルを指定することができます。これを行なうには、attachmentsに対してurlを呼び出し、その結果をimage_tagメソッドに渡します。

```ruby
<p>Hello there, this is our image</p>

<%= image_tag(attachments['image.jpg'].url) %>
```

単にimage_tagを使うだけではダメで、mailerのファイルでインライン添付の設定をする必要がありました。

これを反映させたサンプルコードが以下です。

```ruby
<%= link_to image_tag(attachments['twitter.jpg'].url), twitter_share_url(text: 'アルバムを購入しました', url: artist_artwork_url(@artwork.artist, @artwork), hashtags: 'example', via: 'example') %>
```

```ruby
<%= link_to image_tag(attachments['facebook.jpg'].url), facebook_share_url(u: artist_artwork_url(@album.artist, @album.artwork)) %>
```

オプションの付け方、`link_to`と`image_tag`の組み合わせ方に注意しながら、SNSシェアボタンを記述することができました。

## 最後に

もし本記事に関して修正依頼・ご質問などございましたら[https://twitter.com/o8n_project](https://twitter.com/o8n_project)宛にご連絡いただけると幸いです。

## 参考文献

[シェアボタン ソーシャルプラグイン Facebook公式](https://developers.facebook.com/docs/plugins/share-button?locale=ja_JP)

[Guides | Docs | Twitter](https://developer.twitter.com/en/docs/twitter-for-websites/tweet-button/overview)

[主要 SNS(Twitter, Facebook, LINE)への共有リンクの設定方法](https://r17n.page/2019/11/03/html-share-links-to-major-sns/)

[image_tagメソッドを使ったイメージタグの作成](https://www.javadrive.jp/rails/template/index11.html#section5)

[Rails - 名前付きルートにおける_pathと _urlの違いと使い分け](https://forest-valley17.hatenablog.com/entry/2018/10/17/151935)

[What is the difference between _url and _path while using the routes in rails](https://stackoverflow.com/questions/2350539/what-is-the-difference-between-url-and-path-while-using-the-routes-in-rails)

[^1]:[ActionMailerのビューに画像を追加する](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E3%83%93%E3%83%A5%E3%83%BC%E3%81%AB%E7%94%BB%E5%83%8F%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)

[^2]:[ActionMailerのビューでURLを生成する](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E3%83%93%E3%83%A5%E3%83%BC%E3%81%A7url%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)

[^3]:[RailsのルートURL](https://railstutorial.jp/chapters/filling_in_the_layout?version=5.1#sec-rails_routes)

[^4]:[ActionMailerの全メソッド](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E5%85%A8%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89)
