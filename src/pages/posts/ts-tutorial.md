---
title: "TypeScript入門する"
pubDate: 2020-05-05
draft: false
tags: ["TypeScript", "React"]
layout: ../../layouts/MarkdownPostLayout.astro
---

GWを利用してReactを勉強していたのですが、TypeScriptとの親和性があるということを聞いたので入門してみます。
<!--more-->

# 始める前に

筆者はJavaScript(ES6)について、ググりながら・ドキュメントを読みながらであれば理解できるレベルです。よく[jsprimer](https://jsprimer.net/)を参考にしています。JSでなくてもプログラミング言語に一つでも習熟していれば大丈夫でしょう。

# 用いる教材

- Microsoftのfrontend-bootcamp [https://github.com/Microsoft/frontend-bootcamp](https://github.com/Microsoft/frontend-bootcamp)
  - HTML, CSS, JS, React, Redux, TSをハンズオンで学べる教材です。Microsoft製品風のTodoリストを作ります。Hooksは出てきません。
- TypeScript Deep Dive [https://typescript-jp.gitbook.io/deep-dive/](https://typescript-jp.gitbook.io/deep-dive/)
  - 有志の方によって運営されているオープンソースのTS解説ドキュメントです。

# TypeScript事始め

JSとの相違を意識してポイントを押さえていきます。

## Types（型）

TSを使う一番の醍醐味。

基本の型としてstring, number, booleanなどが指定できます。

```ts
// type.ts

let color: string = 'pink';
let member: string = `譜久村聖のメンバーカラーは ${color}`;

console.log(member)

```

コンパイルしてjsに変換します

```sh
tsc type.ts
# type.jsが生えます
node type.js
# `譜久村聖のメンバーカラーはピンクです`
```

[https://github.com/microsoft/frontend-bootcamp/blob/master/step2-01/demo/src/types/index.ts](https://github.com/microsoft/frontend-bootcamp/blob/master/step2-01/demo/src/types/index.ts)

## インターフェース

typeとよく似ている感じがします。interfaceはクラスやオブジェクトの規格を定義するそうです。公式ドキュメントの冒頭にこう書いてあります。

>TypeScriptの中心的な原則の1つは、型チェックが値の形に焦点を当てることです。 これは、「ダックタイピング」または「構造サブタイピング」と呼ばれることもあります。 TypeScriptでは、インターフェイスはこれらの型に名前を付ける役割を果たし、コード内のコントラクトやプロジェクト外のコードとのコントラクトを定義する強力な方法です。

[https://www.typescriptlang.org/docs/handbook/interfaces.html](https://www.typescriptlang.org/docs/handbook/interfaces.html)

「ダックタイピング」という言葉、『オブジェクト指向設計実践ガイド（技術評論社）』という本を読んで聞いたことがあります。動的型付けオブジェクト指向言語に特徴的な型付け作法で、オブジェクトに何ができるかはオブジェクトそのものが決定するという考えを持っています。Rubyで簡単に見てみます。

```rb
def main(arg)
   puts arg.method
end

class Musume
  def method
    puts 'morning musume'
  end
end

class Juice
  def method
    puts 'Juice=Juice'
  end
end

main(Musume.new)
main(Juice.new)

# => 'morning musume'
# => 'Juice=Juice'
```

ここで、MusumeとJuiceには継承関係がないです。オブジェクト自身がオブジェクトを実行していることがわかります

一方、静的型付け言語の場合は**オブジェクトが、あるインタフェースの全部のメソッドを持っている場合、オブジェクトはそのインタフェースを実行時に実装しているとみなす** と言い換えられます。なるほど？

```ts
interface UpFront {
  age: number;
  color: string;
  group: string;
  name: string;
}

const member: UpFront = {
  age: 23,
  color: 'pink',
  group: 'モーニング娘。20',
  name: '譜久村聖'
}

console.log('member:', JSON.stringify(member))

// member: {"age": 23,"color":"pink","group":"モーニング娘。20","name":"譜久村聖"}
.
```

## ジェネリクス

ジェネリクスが役立つ理由は関数の引数・戻り値の間やクラスのメソッドやインスタンスメンバの間で意味のある型制約を提供することです

```ts
const myStack = new Stack<number>();
myStack.push(1);
myStack.push(2);
console.log('Number on top of the stack:', myStack.pop());
// Number on top of the stack: 2
.
```

Ref
[https://typescript-jp.gitbook.io/deep-dive/type-system/generics#jenerikusugenerics](https://typescript-jp.gitbook.io/deep-dive/type-system/generics#jenerikusugenerics)

## スプレッド演算子、分解

スプレッド演算子は配列・オブジェクトの要素を展開する際に用いられます。

```ts
const obj1 = {
  first: 'シーツ',
  second: '藤本',
}

const obj2 = {
  center: '赤星',
  catcher: '矢野'
}

const megaObj = { ...obj1, ...obj2}

const { first, second, catcher } = megaObj
console.log('first:', first)
console.log('second:', second)
console.log('catcher:', catcher)

// first: シーツ
// second: 藤本
// catcher: 矢野
.
```

JavaScriptで関数に引数を渡す際、`Function.prototype.apply`オブジェクトを使う必要があります。applyメソッドは第一引数にthisとする値を指定、第二引数に関数の引数を配列として渡します。
<br>

Ref
 [Function.prototype.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

```js
function foo(x, y, z) { }
var args = [0, 1, 2];
foo.apply(null, args);

```

thisとする値が指定されていない場合、第一引数はnullになります。

このような時、TypeScriptではスプレッド演算子(...)を使うことでnullを渡すことなくクールに引数を渡せます。

```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo(...args);

```

分解(Destructuring)はオブジェクトまたは配列からプロパティを取り出す簡潔な方法です。

```ts
const obj = { foo: 1, bar: 2 };
const { foo, bar } = obj;
// foo = 1, bar = 2

const arr = [1, 2];
const [foo, bar] = arr;
// foo = 1, bar = 2
.
```

Ref [https://typescript-jp.gitbook.io/deep-dive/future-javascript/spread-operator](https://typescript-jp.gitbook.io/deep-dive/future-javascript/spread-operator)

## Promise, async/await

非同期(async)処理はコードを順番に処理しますが一つの非同期処理が終わるのを待たず次の処理を行います。C#の構文に[影響]を受けたそうです。

ES2015で非同期処理の結果を表現する`Promise`、ES2017で`Async Function`が導入されました

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction

async functionとして定義した関数は必ずPromiseインスタンスを返します。またasync functionの関数内ではawait式を利用できます。PromiseインスタンスがFulfilledまたはRejectedになるまでその場で非同期処理の完了を待ちます。

```ts
function makePromise(): Promise<number> {
  return Promise.resolve(5)
}

async function run() {
  const result = await makePromise();
  console.log('makePromise returned:', result);
}

run();

// makePromise returned: 5
.
```

[影響]: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/

よし、完全に理解しました。

試しに再帰でフィボナッチ数列を作ってみます。

```ts
// fib.ts

export function fib(n: number) {
  return n <= 1 ? n : fib(n-1) + fib(n-2)>
}

const Fibconst: number = 6;
export default Fibconst;

// index.ts
import Fibconst, { fib } from './fib';
console.log(fib(Fibconst))
// 8
.
```

Goもそうですが、静的型付けの言語を身に付けたいと去年から思っていました。VSCodeがヒントを教えてくれるのが良いですね。

非同期処理をちゃんと理解できていないので、実際にプロダクトを作って理解していきたいです。

以下は上の資料を見て作ったTodoリストです。

https://github.com/okamotchan/ms-todo
