---
title: "Check xcode-select when you cannot install db gem (macOS)"
layout: '../../layouts/MarkdownPostLayout.astro'
pubDate: 2020-04-25T18:04:14+09:00
draft: false
share: true
tags: ['Error']
---

Ruby(Sinatra)でAPIを書くためにGemfileを作成しbundle installした際、sqlite3もpgもmysql2もインストールできなかった。

# TL;DR

`sudo xcode-select -switch /`

Ref: [https://github.com/Homebrew/legacy-homebrew/issues/23500](https://github.com/Homebrew/legacy-homebrew/issues/23500)

<!--more-->

## 環境

MacOS Mojave 10.14.6

rbenv 1.1.2

Ruby2.5.1

## アプリ作るぞ〜

```sh
$ bundle init
Writing new Gemfile to /path/to/project_name/Gemfile
```

Gemfileに必要なgemを追加し、bundle install

```Gemfile
source "https://rubygems.org"

gem "rake"
gem "sinatra"
gem "activerecord"
gem "sinatra-activerecord"
gem "sqlite3"
```

するとsqliteだけインストール失敗します。mysql2やpgに関しても同様です。以下エラーログ。

```bash
.
.
Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

中略

To see why this extension failed to compile, please check the mkmf.log which can be found here:

  /Users/username/sinatra_api/vendor/bundle/ruby/2.5.0/extensions/x86_64-darwin-18/2.5.0-static/sqlite3-1.4.2/mkmf.log

extconf failed, exit code 1

Gem files will remain installed in /Users/username/20dev/sinatra_api/vendor/bundle/ruby/2.5.0/gems/sqlite3-1.4.2 for inspection.
Results logged to /Users/username/20dev/sinatra_api/vendor/bundle/ruby/2.5.0/extensions/x86_64-darwin-18/2.5.0-static/sqlite3-1.4.2/gem_make.out

An error occurred while installing sqlite3 (1.4.2), and Bundler cannot continue.
Make sure that `gem install sqlite3 -v '1.4.2' --source 'https://rubygems.org/'` succeeds before bundling.

In Gemfile:
  sqlite3
```

 `/Users/username/sinatra_api/vendor/bundle/ruby/2.5.0/extensions/x86_64-darwin-18/2.5.0-static/sqlite3-1.4.2/mkmf.log`を見に行けと言われています。

```bash
"pkg-config --exists sqlite3"
| pkg-config --libs sqlite3
=> "-lsqlite3\n"
..中略
xcrun: error: invalid active developer path (/Applications/Xcode.app/Contents/Developer), missing xcrun at: /Applications/Xcode.app/Contents/Developer/usr/bin/xcrun
checked program was:
/* begin */
1: #include "ruby.h"
2: 
3: int main(int argc, char **argv)
4: {
5:   return 0;
6: }
/* end */
```

xcode絡みのエラーっぽい。

`missing xcrun at: /Applications/Xcode.app/Contents/Developer/usr/bin/xcrun`を検索にかけると一発目に出てきたのがこれ。

[https://stackoverflow.com/questions/52459194/missing-xcrun-ever-since-upgrading-to-mojave-while-trying-to-use-storyboard-comp](https://stackoverflow.com/questions/52459194/missing-xcrun-ever-since-upgrading-to-mojave-while-trying-to-use-storyboard-comp)

mysql2がインストールできない時`xcode-select --install`したら直ることがあったけど、今回は違った。

もう少し調べてみるとhomebrewのissueがヒット。

[https://github.com/Homebrew/legacy-homebrew/issues/23500](https://github.com/Homebrew/legacy-homebrew/issues/23500)

`sudo xcode-select -switch`

を実行してbundle installすると正常にdbのgemがインストールできました。

おまけ：コマンド履歴

![.](/static/images/post/rireki.png)

途中でopensslのバージョンを変えたり、Rubyのバージョンを変えたりしています。
