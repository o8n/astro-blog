---
title: "RubyにはString#to_aがない"
pubDate: 2020-04-06T10:00:47+09:00
draft: false
share: true
tags: ['Ruby']
images: ["static/images/post/okamotchan.png"]
layout: ../../layouts/MarkdownPostLayout.astro
---

ミニツクRubyというサイトで問題を解いていたところこんなのが。

<!--more-->

>**演習**
以下のコードがコメント部分の通りの出力となるように、「clever_print」メソッドの定義を書いてください。
```ruby
clever_print(["Ruby"], "the", ["Programming", "Language"])
#=> Ruby the Programming Language
 
clever_print(["Agile", "Web", "Development"], "with", { :Rails => 3.0 })
#=> Agile Web Development with Rails 3.0
```

[http://www.minituku.net/courses/500228005/contents/762982128.html](http://www.minituku.net/courses/500228005/contents/762982128.html)

方針としては配列、文字列、ハッシュを引数にとり、それto_aでArrayに変換して共通の配列オブジェクトにまとめ出力するようにすれば良さそうです。

これの模範解答がRuby1.n(?)時代の解答なんです。

```ruby
def clever_print(*args)
  converted = []
  args.each { |arg| converted << arg.to_a }
  puts converted.join(' ')
end

clever_print(%w[hoge foo], 'with', fuga:1)
#=> NoMethodError: undefined method `to_a' for "with":String
```

Ruby2.4を想定して（配列%wしたりhashの書き方を見ていただくと）引数を記述すると、String#to_aメソッドが存在しないので怒られます。

余談ですけど、繰り返し文内でブロック変数のargをargsにすると…

```ruby
def clever_print(*args)
  converted = []
  args.each { |arg| converted << args.to_a }
  puts converted.join(' ')
end

clever_print(%w[hoge foo], 'with', fuga: 1)
#=> hoge foo with {:fuga=>1} hoge foo with {:fuga=>1} hoge foo with {:fuga=>1}
```

…余談でした

今風に書くとこうなる。

```ruby
# frozen_string_literal: true

def method(*args)
  arr = []
  args.map! do |arg|
    if arg.instance_of?(Hash)
      arg.each do |k, v|
        arr << k
        arr << v
      end
    elsif arg.instance_of?(Array)
      arg.each do |v|
        arr << v
      end
    else
      arr << arg
    end
  end
  puts arr.join(' ')
end

method(%w[hoge foo], 'with', fuga: 1)
#=> hoge foo with fuga 1
```

`map`は`collect`でも良いです

`Object#instance_of?`により引数がHashの場合はkeyとvalueを予め用意した配列に格納、Arrayの場合
はvalueを格納、文字列の場合はそのまま文字列を格納。

少し冗長になってしまいますがシンプルにかけます。
