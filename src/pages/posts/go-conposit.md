---
title: "Goのコンポジット型"
pubDate: 2020-10-13
draft: false
share: true
images: ["static/images/post/blog.png"]
tags: ["Go"]
layout: ../../layouts/MarkdownPostLayout.astro
---

Goにはコンポジット型という物がある。配列、スライス、マップ、構造体のこと。

Goでは配列ではなくスライスを使う局面が多い。

```go

var a [3] int // 3個の整数の配列
fmt.Println(a[0]) // 最初の要素

// インデックスと要素の表示
// キーとバリューのようなものですね
for i, v := range a {
  fmt.Printf("%d %d\n", i, v)
}

// 要素だけを表示する
for _, v := range a {
  fmt.Printf("%d\n", v)
}
```
