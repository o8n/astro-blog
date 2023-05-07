---
title: "DockerforMacのファイルI/Oの処理速度が遅く感じる"
pubDate: 2020-04-02
draft: false
tags: ["docker"]
layout: ../../layouts/MarkdownPostLayout.astro
---

Docker for Mac、使い物になっていますか。私はなっていません。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">チーム開発してるRailsアプリをdocker-compose上でやっていくのレスポンスが遅かったりデバッグが辛いって聞いたんですけど実際どうなんでしょうか、私は知見を求めています</p>&mdash; o8n (@o8n_project) <a href="https://twitter.com/o8n_project/status/1245028278596800514?ref_src=twsrc%5Etfw">March 31, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<!--more-->

Railsアプリの開発環境をdockerに載せてみたんですけど、最初railsサーバーの起動がすごく遅くて使いもんにならないなと思いました。でもcachedオプション利用したり、volumeを工夫すればマシになるんですね
　　
# volume mounts&cached

[https://docs.docker.com/compose/rails/](https://docs.docker.com/compose/rails/) を参考にしました。

```Dockerfile
FROM ruby:2.4.2

RUN apt-get update -qq && \
    apt-get install -y nodejs postgresql-client
RUN mkdir /app
WORKDIR /app
COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock
RUN bundle install
COPY . /app

# Add a script to be executed every time the container starts.
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Start the main process.
CMD ["rails", "server", "-b", "0.0.0.0"]
```

docker-compose.ymlをみてみます

``` docker-compose.yml
version: '3'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/app
    ports:
      - "3000:3000"
    depends_on:
      - db
```

ボリューム、ぼくは「コンテナ内でデータを永続化させるためのメカニズム」という風に理解をしています。以下を参照。

[https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)


レスポンスを早くするためにweb要素のボリュームを工夫します。`tmp`とか`vendor`とか、必要なさそうなディレクトリをvolumeでマウント上書きします。さらにディレクトリ全体に対して`cached`オプションを適用します

``` docker-compose.yml
version: '3'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/app:cached
      - /app/vendor
      - /app/tmp
      - /app/log
      - /app/.git
    ports:
      - "3000:3000"
    depends_on:
      - db
```

キャッシュとボリュームマウントに関して、公式ドキュメントをあたってみました。

>You can configure container-and-host consistency requirements for bind-mounted directories in Compose files to allow for better performance on read/write of volume mounts. These options address issues specific to osxfs file sharing, and therefore are only applicable on Docker Desktop for Mac.

>Composeファイルでバインドマウントされたディレクトリのコンテナとホストの整合性要件を設定することで、ボリュームマウントの読み書きのパフォーマンスを向上させることができます。これらのオプションはOSXFSファイル共有に特有の問題に対応しているため、Docker Desktop for Macにのみ適用されます。

  原文

  [https://docs.docker.com/compose/compose-file/#caching-options-for-volume-mounts-docker-desktop-for-mac](https://docs.docker.com/compose/compose-file/#caching-options-for-volume-mounts-docker-desktop-for-mac)

volumeに関してはこちらが詳しいです。
[https://docs.docker.com/docker-for-mac/osxfs-caching/](https://docs.docker.com/docker-for-mac/osxfs-caching/)

正直、原文読んでも何が起こってるか詳しくは理解できなかったです…。分かったことは、Mac特有の問題はcachedオプションとvolumeマウントをチューニングしたらマシになるということです。

![.](/static/images/post/rireki.png)
