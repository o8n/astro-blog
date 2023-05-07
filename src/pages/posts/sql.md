---
title: "20200425 sql"
pubDate: 2020-04-25
draft: false
tags: ["SQL"]
layout: ../../layouts/MarkdownPostLayout.astro
---

Ruby on Railsを機にエンジニア人生をスタートしてると、ActiveRecordのおかげで生SQLとかDBについて全然基本知識がないまま大人になってしまいます。

まずいなと思ったのでSQLとDBについて勉強していこうと思います。

<!--more-->

基本情報技術者試験とかDBスペとかにも出るようですし。

（今春受ける予定だったけど中止になった）

まずは気になる用語から。

## Index Scan/Seq Scan

索引を利用してレコードを読み取るのをIndex Scan、全文検索をSeq Scanという（超絶雑意訳）

[PostgreSQL 第11章 インデックス](https://www.postgresql.jp/document/11/html/indexes.html)

[PostgreSQL 第12章 全文検索](https://www.postgresql.jp/document/11/html/textsearch.html)

## 同時実行制御

[PostgreSQL 第13章 同時実行制御](https://www.postgresql.jp/document/11/html/mvcc.html)

## N+1問題

SQLクエリがデータ量N+1回走って取得するデータが多くなるにつれてパフォーマンスが低下する問題。

```sh
Processing by PostsController#index as HTML
  Post Load (0.2ms) SELECT "posts".* FROM "posts"
  User Load (0.2ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 1]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 2]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 3]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 4]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 5]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 6]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 7]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 8]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 9]]
  User Load (0.1ms) SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1  [["id", 10]]
  Rendered posts/index.html.erb within layouts/application (32.9ms)
Completed 200 OK in 147ms (Views: 132.6ms | ActiveRecord: 2.0ms)
```

上ではユーザーひとりに対して1クエリ叩いている。クエリ2回だけ発行するようにする。

```sh
Processing by PostsController#index as HTML
  Post Load (0.2ms)  SELECT "posts".* FROM "posts"
  User Load (0.2ms)  SELECT "users".* FROM "users"  WHERE "users"."id" IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  Rendered posts/index.html.erb within layouts/application (26.2ms)
Completed 200 OK in 138ms (Views: 124.4ms | ActiveRecord: 1.1ms)
```

N+1解決法として、Eager Loadingという方法があります。

|メソッド   |キャッシュ|クエリ     | 用途         |
|---------:|:-------:|:--------:|:------------|
|joins     |しない    |単数      |絞り込み          |
|eager_load|する      |単数      |キャッシュと絞り込み|
|preload   |する      |複数      |キャッシュ     |
|includes  |する      |場合による |キャッシュ、必要なら絞り込み|

<br>

#### 参考

[https://ruby-rails.hatenadiary.com/entry/20141108/1415418367](https://ruby-rails.hatenadiary.com/entry/20141108/1415418367)
[https://qiita.com/k0kubun/items/80c5a5494f53bb88dc58](https://qiita.com/k0kubun/items/80c5a5494f53bb88dc58)

## Redis(インメモリDB)

## トランザクション、ロールバック


## SQLインジェクション

>アプリケーションのセキュリティ上の不備を意図的に利用し、アプリケーションが想定しないSQL文を実行させることにより、データベースシステムを不正に操作する攻撃方法のこと。また、その攻撃を可能とする脆弱性のことである

[https://ja.wikipedia.org/wiki/SQL%E3%82%A4%E3%83%B3%E3%82%B8%E3%82%A7%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3](https://ja.wikipedia.org/wiki/SQL%E3%82%A4%E3%83%B3%E3%82%B8%E3%82%A7%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)

挿入された値によって元のSQLが実行されるのにくわえ別のSQLも実行されるというもの。これの対策として

- 引用符をエスケープして引用符によって引用符内の文字列が途中で終了するのを阻止する
- プリペアードステートメント
- ストアドプロシージュア
- （ORMマッパーの利用）

などがあげられる。

ユーザーには値の入力は許可してもコードの入力は許可してはならない。

とSQLアンチパターンに書いてある。

随時更新する
