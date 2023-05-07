---
title: "Dockerでコンテナをたてる"
pubDate: 2020-03-28
draft: false
tags: ["docker"]
layout: ../../layouts/MarkdownPostLayout.astro
---

# Dockerのココがスゴイ！

![.](/static/images/post/horizontal-logo-monochromatic-white.png)

- 環境構築が楽になる。イメージを作成してコンテナで作業することができる
- 開発に使ったコンテナをCIでテストし、CIでテストしたコンテナをサーバーにデプロイすることができる
- 自分の環境では動くけどあの人の環境では動かない、ということが減る。環境に依存せずに開発ができる

<!--more-->

## イメージとコンテナの関係
  
|名称|役割|
|---|---|
|Dockerイメージ|Dockerコンテナを構成するファイルシステムや実行するアプリケーションや設定をまとめたもの。 |
|Dockerコンテナ|Dockerイメージを元に作成され、具現化されたファイルシステムとアプリケーションが実行されている状態|
  
<br>
<br>

Dockerの基本操作は「イメージ(image)」に関する操作と「コンテナ(container)」に関する操作に大別されます。DockerイメージはDockerコンテナを作成するためのテンプレートです。Dockerfileはイメージを構築するための手順書のようなものです。
  
<br>
Dockerコンテナは実行中・停止・破棄の3つの状態に分類することができます。これはコンテナのライフサイクルと呼びます。
<br>

# 実践 docker-compose
  

実運用中のRailsアプリケーションの開発環境をdocker-composeで動かせるようにしたのでそこで得た知見を少し共有します。
  

## イメージ作成・コンテナ作成、起動

```sh
docker-compose up
```

## DB作成

```sh
docker-compose run web rails db:create

docker-compose run web rails db:migrate
```

## railsサーバー起動

```sh
docker-compose run web rails s
```

## コンテナ停止

```sh
docker-compose down
```

## その他コマンド

```sh
# 起動しているコンテナの一覧
docker ps

# 起動中・停止中含む全てのコンテナを表示
docker ps -a

# 全てのコンテナを停止する
docker stop $(docker ps -q)

# 全てのコンテナを削除する
docker rm $(docker ps -q -a)

# 全てのイメージを削除する
docker rmi $(docker images -q)
```

## 参考

[Docker/Kubernetes 実践コンテナ開発入門](https://gihyo.jp/book/2018/978-4-297-10033-9)
