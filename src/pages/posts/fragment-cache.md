---
title: "フラグメントキャッシュについて"
pubDate: 2020-10-14
draft: false
tags: ["Web", "Rails"]
layout: ../../layouts/MarkdownPostLayout.astro
---

Webアプリにおけるキャッシュ、一回アクセスしたコンテンツを保存して同じリクエストがきたら先ほど表示したコンテンツを再利用するようにする仕組みです。Ruby on Railsには面倒な設定不要で使えるキャッシュ機構が用意されております

<!--more-->

[https://railsguides.jp/caching_with_rails.html](https://railsguides.jp/caching_with_rails.html)

[https://api.rubyonrails.org/classes/ActionView/Helpers/CacheHelper.html](https://api.rubyonrails.org/classes/ActionView/Helpers/CacheHelper.html)

[https://techracho.bpsinc.jp/hachi8833/2017_05_23/40237](https://techracho.bpsinc.jp/hachi8833/2017_05_23/40237)

フラグメントキャッシュと向き合った1週間だったので簡単に書き綴らせてください。ドキュメントはRails6の内容だったりしますが、自分はRails5.1.6で動作確認をしています。多分両者間で大した差はないと思います

## cache

キャッシュにはページキャッシュ、アクションキャッシュなどいくつか種類がありますが、Rails開発で使うのはフラグメントキャッシュとロシアンドールキャッシュがメインではないかと思っています。

 ページ内で個別にproductをキャッシュしたい場合。

``` ruby
<% @products.each do |product| %>
  <% cache product do %>
    <%= render product %>
  <% end %>
<% end %>
```
  
リクエストを送るとキャッシュが生成されます。コントローラーで以下のように記述すると生成されたキーがログに出力されます。
  
``` md
def index
  @product = @products.cache_key
  logger.info ' キーが表示される #{@product} '
end

キーが表示される products/query-ef4bd7c1abfbab2a6b962c0f42703357-12-20200316073110500878
```

キーの中間にある長い数字はこの場合product_idと、productレコードのupdated_at属性のタイムスタンプ値です。updated_at値が更新されると新しいキーが生成され、そのキーで新しいキャッシュを保存します。

フラグメントキャッシュを適用している範囲内でさらにフラグメントをキャッシュしたいことがあります。この時にロシアンドールキャッシュという手法をとります。以下で親テーブルproduct, 子テーブルgameを想定します

レンダリング

``` ruby
<% cache product do %>
  <%= render product.games %>
<% end %>
```
  
``` ruby
<% cache game do %>
  <%= render game %>
<% end %>
```
  
gameが更新されると`updated_at`が更新されますが、productのupdated_atは変更されないので、productのキャッシュは期限切れにならず古いデータを返します。
  
gameが更新されたらproductのupdateも更新して欲しいので、`touch: true`でテーブルを関連付けます。

``` ruby
class Product < ApplicationRecord
  has_many :games
end

class Game < ApplicationRecord
  belongs_to :product, touch: true
end 
```
  
参考
  
[belongs-toのオプション-touch （Railsガイド）](https://railsguides.jp/association_basics.html#belongs-to%E3%81%AE%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3-touch)

## 開発環境をキャッシュモードに
  
`rails dev:cache` をコンソールで実行すれば開発環境でキャッシュモードの切り替えができます。ブラウザ側でキャッシュを削除したりする必要はないです。
  
キャッシュモード解除したい場合はもう一度上のコマンドを実行する、もしくは`bundle exec rails r 'Rails.cache.clear'`を実行することでキャッシュを解除することもできます。

## キャッシュの効果
  
キャッシュって本当に効果があるのか？と疑問に感じたので速度を計測しました。計測に使用したのはrack-mini-profilerというgemです。使用法については以下をご覧ください。

[https://github.com/MiniProfiler/rack-mini-profiler](https://github.com/MiniProfiler/rack-mini-profiler)

画像とテキストのカードを一覧表示（1ページにつき15枚程度）するページだったのですが、キャッシュ有無により180msほどの速度差が見られました。確かに効果はある。(でもログを見たら無駄なSQLが走っていたのでそれはまた別問題）

## 参考

[https://qiita.com/katsumata_ryo/items/18d1686d99cbfaff8a8d](https://qiita.com/katsumata_ryo/items/18d1686d99cbfaff8a8d)

なお本記事は

[https://nikogory.hatenablog.com/entry/2020/03/20/232911](https://nikogory.hatenablog.com/entry/2020/03/20/232911)

より移行したものです。
