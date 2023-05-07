---
title: "[Ruby] openssl 1.0.0 LoadError"
pubDate: 2020-09-08
draft: false
tags: ["Ruby", "Rails"]
layout: ../../layouts/MarkdownPostLayout.astro
---

rbenvでRubyのバージョン切り替えをしており、Ruby2.6→2.4に切り替えた後、Rails serverを起動したときに発生したエラー。過去にも一度遭遇した記憶がある。その時はbrew install opensslをして乗り切ったと思うけど、今回はそれで解決しなかった。

## TL;DR

Let's try

```sh
rbenv install 2.4.2 #(your using version)
```
<!--more-->

## My env

OS：macOS Catalina 10.15.5

パッケージ管理：Homebrew 2.5.0-2-g17d92a2

Rubyバージョン管理：rbenv 1.1.2

Rubyバージョン：ruby 2.4.2p198 (2017-09-14 revision 59899) [x86_64-darwin18]

状況：railsコマンド全般使えない

## Error Message

```sh
$ rails server

/Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/2.4.0/openssl.rb:13:in `require': dlopen(/Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/2.4.0/x86_64-darwin18/openssl.bundle, 9): Library not loaded: /usr/local/opt/openssl/lib/libssl.1.0.0.dylib (LoadError)
  Referenced from: /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/2.4.0/x86_64-darwin18/openssl.bundle
  Reason: image not found - /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/2.4.0/x86_64-darwin18/openssl.bundle
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/2.4.0/openssl.rb:13:in `<top (required)>'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/activesupport-5.1.6/lib/active_support/key_generator.rb:2:in `require'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/activesupport-5.1.6/lib/active_support/key_generator.rb:2:in `<top (required)>'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/application.rb:4:in `require'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/application.rb:4:in `<top (required)>'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails.rb:12:in `require'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails.rb:12:in `<top (required)>'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/commands/server/server_command.rb:4:in `require'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/commands/server/server_command.rb:4:in `<top (required)>'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command/behavior.rb:82:in `require'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command/behavior.rb:82:in `block (2 levels) in lookup'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command/behavior.rb:78:in `each'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command/behavior.rb:78:in `block in lookup'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command/behavior.rb:77:in `each'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command/behavior.rb:77:in `lookup'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command.rb:68:in `find_by_namespace'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/command.rb:42:in `invoke'
	from /Users/okamotomasahiro/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/railties-5.1.6/lib/rails/commands.rb:16:in `<top (required)>'
	from bin/rails:4:in `require'
	from bin/rails:4:in `<main>'
```

## I tried

homebrewを使ってopensslをインストールしてみる。前もこれで直った記憶がある。

```sh
$ brew install openssl

🍺  /usr/local/Cellar/openssl@1.1/1.1.1g: 8,059 files, 18MB
```

check

```sh
$ brew list -1 | grep openssl

openssl@1.1
```

`openssl@1.0` is deprecated...

homebrewのフォーラムを発見しました。

[today I tried to install a new Ruby version through RVM, which triggered a brew update where a new openssl@1.1 version was installed and now I am unable to](https://discourse.brew.sh/t/brew-update-breaks-openssl/6584/7)

使っているRubyとOpenSSL1.1が互換性がない、OpenSSL1.0はdeprecateしててbrew installできなくて困っているという内容です。私と同じ状況です。

この中でrvmのissueが紹介されています。

[OpenSSL 1.1 via Homebrew satisfies autolibs and builds ruby 2.3 but built rubies throw LoadError when attempting to use openssl](https://github.com/rvm/rvm/issues/4819)

解決法として

```sh
brew install rbenv/tap/openssl@1.0
rvm install 2.3.8 -C --with-openssl-dir=`brew --prefix openssl@1.0`
```

が示されています。ですが、私はrbenvを使っているのでこの策は該当しません。

とりあえず

```sh
brew install rbenv/tap/openssl@1.0
```

を実行すると正常にopenssl@1.0が手に入りました。確認してみます。

```sh
$ brew list -1 | grep openssl

openssl@1.0
openssl@1.1
```

手に入りました。

rvm installは実行しても意味がないので

```sh
rbenv install 2.4.2
```

を実行します。この時、uninstallしたりする必要はありません。既にrbenvで管理しているRubyをインストールし直しただけという形になります。

```sh
rbenv: /Users/okamotomasahiro/.rbenv/versions/2.4.2 already exists
continue with installation? (y/N) y
Downloading openssl-1.1.0j.tar.gz...
-> https://dqw8nmjcqpjn7.cloudfront.net/31bec6c203ce1a8e93d5994f4ed304c63ccf07676118b6634edded12ad1b3246
Installing openssl-1.1.0j...
Installed openssl-1.1.0j to /Users/okamotomasahiro/.rbenv/versions/2.4.2

Downloading ruby-2.4.2.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.2.tar.bz2
Installing ruby-2.4.2...

WARNING: ruby-2.4.2 is past its end of life and is now unsupported.
It no longer receives bug fixes or critical security updates.

ruby-build: using readline from homebrew
Installed ruby-2.4.2 to /Users/okamotomasahiro/.rbenv/versions/2.4.2
```

オッ。rbenv2.4.2にopenssl-1.1.0jがインストールされたようです。

ちなみにRuby2.4は2020年3月31日を持ってサポートを終了しています。

[https://www.ruby-lang.org/ja/news/2020/04/05/support-of-ruby-2-4-has-ended/](https://www.ruby-lang.org/ja/news/2020/04/05/support-of-ruby-2-4-has-ended/)

この状態からRailsアプリケーションを動かすためにrailsサーバーを起動すると無事に作動しました。

### おまけ

無事に作動する前に`bundle install`を実行しました。

その後、rails serverを実行すると

```sh
psql: error: could not connect to server: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

これもよく見るエラー（Postgres特有のエラーです）。`brew info postgresql`で状況を確認します。

```sh
postgresql: stable 12.4 (bottled), HEAD
Object-relational database system
https://www.postgresql.org/
/usr/local/Cellar/postgresql/12.4 (3,223 files, 37.8MB) *
  Poured from bottle on 2020-09-08 at 21:52:46
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/postgresql.rb
License: PostgreSQL
==> Dependencies
Build: pkg-config ✔
Required: icu4c ✔, krb5 ✔, openssl@1.1 ✔, readline ✔
==> Options
--HEAD
	Install HEAD version
==> Caveats
To migrate existing data from a previous major version of PostgreSQL run:
  brew postgresql-upgrade-database

This formula has created a default database cluster with:
  initdb --locale=C -E UTF-8 /usr/local/var/postgres
For more details, read:
  https://www.postgresql.org/docs/12/app-initdb.html

To have launchd start postgresql now and restart at login:
  brew services start postgresql
Or, if you don't want/need a background service you can just run:
  pg_ctl -D /usr/local/var/postgres start
==> Analytics
install: 257,705 (30 days), 625,283 (90 days), 1,545,309 (365 days)
install-on-request: 250,840 (30 days), 609,026 (90 days), 1,474,422 (365 days)
build-error: 0 (30 days)
```

brew upgradeしたため、Postgresが12.4にバージョンアップされています。今まで11.4を使ってました。指示に従い、brew postgresql-upgrade-database します。

これが成功し、rails serverを起動してみます。するとアプリケーションが作動しました。
