---
title: TypeScriptのジェネリクス
tags: [TypeScript]
createDate: 2023-03-12
updateDate: 2023-03-12
slug: typescript-generics
draft: false
---

## はじめに

今回はTypeScriptのジェネリクスについてです。JavaScriptがある程度かければTypeScriptでつまづく箇所は結構少ないとは思いますが、つまづいたのがジェネリクスという概念でした。  
今回はジェネリクスの仕組みと何を解決できるのかについて調査してみます。  

## ジェネリクスとは

型の安全性とコードの共通化を両立するための仕組み

抽象的な型引数を使用して、実際に利用されるまで型が確定しない`クラス`、`関数`、`インターフェース`を実現するために使われる。

参考サイトの例をみてみます。

中身のロジックが同じで引数の型が異なる関数が以下のように三つあります。

```typescript
function chooseRandomlyString(v1: string, v2: string): string {
  return Math.random() <= 0.5 ? v1 : v2;
}
function chooseRandomlyNumber(v1: number, v2: number): number {
  return Math.random() <= 0.5 ? v1 : v2;
}
function chooseRandomlyURL(v1: URL, v2: URL): URL {
  return Math.random() <= 0.5 ? v1 : v2;
}
```

これらを共通化するにはどうすればいいでしょうか。  
解決策の1つは`any`型を使用して型チェックを放棄することですがせっかくTypeScriptを使う以上避けたいです。

ここで登場するのがジェネリクスで、下記のように書けます。

```typescript
function chooseRandomly<T>(v1: T, v2: T): T {
  return Math.random() <= 0.5 ? v1 : v2;
}
chooseRandomly<string>("勝ち", "負け");
chooseRandomly<number>(1, 2);
chooseRandomly<URL>(urlA, urlB);
```

ここで、`T`には任意の型変数名が入ります。つまり、なんでも構わない名前です。慣習的に`Type`の`T`が使用されます。

```typescript
function chooseRandomly<T>(v1: T, v2: T): T {
  return Math.random() <= 0.5 ? v1 : v2;
}
let str = chooseRandomly<string>(0, 1);
// エラー
Argument of type 'number' is not assignable to parameter of type 'string'.
str = str.toLowerCase();
```

この例の場合、引数の型によって返り値の型が決まるため、上記のような型のエラーに気づくことができます。

このように型の安全性と汎用性を共存させることができます。

## わからないこと

ジェネリクスがここまでの例で解決することを理解はできましたが、オーバーロードやユニオン型でも解決できそうな気がします。

例を再度考えてみます。

ジェネリクスを使うことで型の種類を問わず引数のように使用する側で渡すことができる点では汎用性は高そうです。

```typescript
function chooseRandomly<T>(v1: T, v2: T): T {
  return Math.random() <= 0.5 ? v1 : v2;
}
```

### ジェネリクスの型推論

ジェネリクスを使用すると型推論されます。

```typescript
function hoge<T>(x: T) {
    alert(x instanceof Date);
}
hoge<string>("new Date()"); // false
hoge<Date>(new Date()); // true
```

このように、ジェネリクスを使って型推論を利用することができ、さらに型の指定はコンパイラに推論可能であれば型名を省略できます。

```typescript
function hoge<T>(x: T) {
    alert(x instanceof Date);
}
hoge("new Date()"); // false
hoge(new Date()); // true
```

### 問題点: 型のメソッドを使えない

ここまでの説明だと型自体を決定することはできるんですが、実際に渡ってくる引数の型は呼び出されるまで分かりません。  
そのために型に属する機能をなにも呼ぶことができない状態になります。  
この問題を解決するためにはジェネリクスを下記のように使用します。  
そしてこの使い方こそがジェネリクスの真価を発揮する方法だと思っています。  

### 解決策: ジェネリクスの制約

```typescript
interface X {
  sayMyName();
}
 
class Y implements X {
  public sayMyName() {
    alert("I'm Big-Boy");
  }
}
 
function a<T>(t: T) {
  t.sayMyName();
}
 
a(new Y());
```

上記のコードはコンパイルできません。理由は`T`という型がコンパイル時には未知の型であるためです。  
この問題を解決するためには型引数`T`自体に制約を追加します。  

```typescript
function a<T extends X>(t: T) {
    t.sayMyName();
}
```

`<T extends X>`で型`X`もしくは`X`を継承した型のみを型引数にとるということを表現します。

ちなみに、下記のように利用したいメンバーだけを強制することもできます。

```typescript
function a<T extends { sayMyName(); }>(t: T) {
    t.sayMyName();
}
```

この使い方はTypeScriptのダックタイピング的な動作を利用しています。  

### ダックタイピング

個人的にこれまで静的型付け言語に触れることが多く、プロダクトで使用しているPHPも現在はほぼC#のような静的型付けに近い運用をしているため`ダックタイピング`という考え方に馴染みがないのでまとめておきます。  

コンパイル時に型の検査をするのではなく、実行時に実行するという方法です。  
オブジェクトに何ができるかはクラスではなく実行時のオブジェクトそのものが決定するという考え方です。

>"If it walks like a duck and quacks like a duck, it must be a duck"
>（もしもそれがアヒルのように歩き、アヒルのように鳴くのなら、それはアヒルに違いない）

言い換えれば、`同じ振る舞いを持つものは共通のインターフェースを持つ`とも言えます。  
インターフェースの判定に継承は関係なく、必要なインターフェースを持っているかどうかのみに着目します。  

ここ最近Ruby on Railsに触れる機会があるのですが、Rubyのばあいは一般的なクラス継承もできるので継承によるポリモーフィズムも利用できます。が、ダックタイピングを使えば継承が不要であり型による制約に縛られることなく簡素なコードで実現ができます。  
ただ、動的型付け言語でダックタイピングは乱用されることがあり、ここ最近では静的型付け言語に注目が集まっているのでメリットとデメリットを理解することが必要です。

## まとめ

- ジェネリクスを使用することで引数の型が決まっていない処理を共通化できる
- コレクション等でメリットが大きい
- ある程度の型推論にも使えるが過信はできない
- 振る舞いを利用する場合は型引数自体に`extends`を使って制限をかける必要がある

### 参考

<https://qiita.com/k-penguin-sato/items/9baa959e8919157afcd4>  
<https://typescriptbook.jp/reference/generics>  
<https://www.buildinsider.net/language/tsgeneric/01>  
<https://ja.wikipedia.org/wiki/%E3%83%80%E3%83%83%E3%82%AF%E3%83%BB%E3%82%BF%E3%82%A4%E3%83%94%E3%83%B3%E3%82%B0>  
