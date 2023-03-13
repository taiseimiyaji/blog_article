---
title: TypeScriptに入門する
tags: [TypeScript]
createDate: 2022-12-25
updateDate: 2022-12-31
slug: typescript-init
---


## はじめに
前回の記事同様直近の大きな課題感を感じているのはフロントエンド技術です。  
今回はその中でも最近主流となっているTypeScriptに入門してみます。  
私のエンジニア人生はC言語から始まっているので静的型付けで書けないJSに違和感をずっと感じていたため、個人開発等で導入し、ゆくゆくは社内のプロダクトもどんどん移行していきたいなと感じています。  
今回は備忘録としてJavaScriptの仕様含めたおさらい記事として残しておきます。  

## TypeScriptとは

- Microsoftによって開発されているAltJSの一種
- AltJSという言葉をあまり聞かなくなるくらい主流になっている
- AltJS=JavaScriptの代替となる言語
- 静的型システムを備えているのが最大の特徴

歴史的には最初のプレビュー版の公開が2012年の10月。  
バージョン1.0の公開、つまり正式リリースが2014年4月。  

### 静的型システムの例

静的型システムとは、ざっくりいうと、変数や式が型をもつということ。

```TypeScript
const str: string = "foobar";
```

変数`str`はstring型を持っているという`型注釈`が書かれている

同時にTypeScriptは`型推論`という機能も充実している。
型推論とは、型注釈を書かなくてもTypeScriptが補って変数などの型を決めてくれる機能。

下記のコードは先程のコードから型注釈を省略したもの。
```TypeScript
const str = "foobar";
```

### 型があるとなぜ嬉しいのか

- 型安全性
- ドキュメント化

型安全性とは、実行前にコンパイラが型チェックして検出してくれるしくみ。
コンパイラとは、プログラミング言語で書かれたコードを機械語に翻訳するための仕組みのこと。

コンパイラは、`構文が正しくない`エラーと`型チェックが失敗したこと`を表す型エラーの２種類のエラーを主に返すが、特にコンパイラによって型エラーを検出できるのが静的型付け言語の恩恵。

型エラーとは？  
型チェックの失敗は型に矛盾が発生した場合に起こる。

```TypeScript
function repeatHello(count: number): string {
    return "hello".repeat(count)
}
// countはint型を期待しているが文字列型が入力されるためエラーが発生
console.log(repeatHello("wow"));
```

ドキュメント化とは型の情報がソースコードに書かれることでプログラムを読解する助けになる、ということ。  
例えば、先程の例を見ると1行目を見るだけで関数`repeatHello`がnumber型の引数を受け取ってstring型の返り値を返すことがわかる。  
適切な関数名、コメントに加えて型の情報があればプログラムの読解時に関数の中身まで読む必要がなく、ある程度中身を推測しながら読み進める事ができる。  
これは特に大規模なシステム開発で効果を発揮する。  

また、型の情報は人間の助けとなるだけでなくコンピュータにとっても助けとなる。IDEによる入力補完の恩恵をより受けられる。  

静的とは、`実際にプログラムを実行しなくても行えるチェック`のこと。プログラムの文面だけを見て行われるチェック。  
反対に動的なチェックとは、`テスト`など実際にプログラムを実行してその結果を見てプログラムに間違いがないかを確認するもの。

TypeScriptはランタイム(実行時)の挙動が型情報に依存しないため、TypeScriptの持つ役割はあくまで静的なチェックのみ。

`ランタイム(実行時)の挙動が型情報に依存しない`具体例  

```TypeScript
function double(value: number) {
    console.log(value * 2);
}

function double(value: string) {
    console.log(value.repeat(2));
}

double(123);
double("hello");
```

上記の様に、同名関数で引数の取る型が違うものを定義できるプログラミング言語が存在する。  
が、TypeScriptでは実行時の挙動は型情報に依存しないため、この機能は存在しない。あくまでTypeScriptはJavaScriptの拡張という立ち位置なので、役割を型チェックに絞ってランタイムの挙動はJavaScriptに従うという思想。　　

### トランスパイル
TypeScriptコンパイラの型チェック以外の役割として、トランスパイルという仕組みがある。

トランスパイルとは、TypeScriptコードをJavaScriptコードに変換するということ。(機械語ではなく他言語への変換なのでトランスパイルと呼ばれるが、単にコンパイルと呼ぶ場合もある)

TypeScript->JavaScriptへのトランスパイルは単に型注釈を取り除くだけの処理が行われる。つまり、TypeScriptは基本的にJavaScriptに型の概念を導入しただけなのでそれを取り除くだけで変換ができる。

### TypeScriptにおけるプリミティブ型

プリミティブとは、「原始的な」という意味ので単語で、プログラミング言語における基本的な値を示す。

TypeScriptには以下のプリミティブ型がある。

- 文字列
- 数値(number)
- 真偽値
- BigInt
- null
- undefined
- シンボル

以降、最もよく使用される数値、文字列、真偽値についてJavaScriptの復習も兼ねて再確認する。

### 数値(number)型

小数と整数の区別なく数値を扱える型。  
TypeScriptで欠かせない要素が`リテラル`という概念。  
リテラルとは、何らかの値を生み出すための式のこと。  

数値リテラルとは、以下の`5`のようなもの

```TypeScript
const value = 5;
```
他にも2進数や8進数、16進数のリテラルが良く使われる。

```TypeScript
const binary = 0b1010; // 2進数リテラル
const octal = 0o755; // 8進数リテラル
const hexadecimal = 0xff; // 16進数リテラル

// 指数表記もリテラルが存在する
const big = 1e8;
const small = 4e-5;
```


### 文字列(string)型

文字列リテラルにはダブルクォートとシングルクォートの２種類の書き方があるが、機能上の違いはない。

```TypeScript

const str1: string = 'Hello,world!';
const str2 = "Hello,world!";

```

加えて、`テンプレートリテラル`ろいうリテラルも存在する。

```TypeScript

const message: string = `Hello
world!`

const str1: string = "Hello";
const str2: string = "world!";

console.log(`${str1},${str2}`); // "Hello,world!と表示"

```

普通の文字列リテラルとの違いは、

- リテラル中で改行が可能である
- 式を文字列の中に埋め込む事ができる

### 真偽値(boolean)型

trueとfalseの２つの値からなる型。

```TypeScript
const no: boolean = false;
const yes: boolean = true;

console.log(yes, no); // true falseと表示される
```

## オブジェクトとは

TypeScriptにおけるオブジェクトは必ずしもクラスに由来するものではない。  
Java等の言語ではクラスとオブジェクトは切っても切り離せない関係にあるので、クラスに触れずにオブジェクトの話ができるTypeScriptはすこしギャップがある。

### オブジェクトは**連想配列**である

オブジェクトはいくつかの値をまとめたデータである。

```TypeScript
const obj = {
    foo: 123,
    bar: "Hello,world!"
};

console.log(obj.foo);
console.log(obj.bar);
```

変数`obj`に代入されている{}を`オブジェクトリテラル`と呼ぶ。  
また、`:`の後ろには固定された数値や文字列を書くだけでなく、変数の値を用いたり、プロパティの値を直接計算したりすることができる。

```TypeScript
const user = {
    name: input ? input : "名無し",
    age: 20,
};
```

オブジェクトリテラルは、`プロパティ名: 変数名`という形の場合、かつプロパティ名と変数名が同じである場合は省略記法を使用することができる。

```TypeScript
const name = input ? input: "名無し";
const user = {
    name: name,
    age: 20,
};
```
```TypeScript
const name = input ? input: "名無し";
const user = {
    name,
    age: 20,
};
```

### オブジェクトに対するconstの制限について

constで宣言された変数に再代入することはできないが、オブジェクトのプロパティの書き換えはconstによって制限されない。

次の例のように、constで宣言された変数自体に別のオブジェクトを再代入する場合はエラーが発生する。

```TypeScript
const user = {
    name: "hoge",
    age: 25,
};

const user = {
    name: "fuga",
    age: 15,
};
```

### スプレッド構文

オブジェクトリテラル中では`スプレッド構文`を使用することができる。

オブジェクトの作成時にプロパティを別のオブジェクトからコピーする事ができる。

```TypeScript
const obj1 = {
    bar: 456,
    baz: 789
};
const obj2 = {
    foo: 123,
    ...obj1
};

// obj2 = {foo: 123, bar: 456, baz: 789}
console.log(obj2);
```

既存のオブジェクトを拡張した別のオブジェクトを作りたい場合に有用。あくまでコピーなので、コピー元のオブジェクトのプロパティを変更してもコピー先には影響しない。

### オブジェクトの等価性

TypeScriptではオブジェクトが暗黙にコピーされることはなく、複数の変数に同じオブジェクトが入る場合が存在する。

```TypeScript
const foo = { num: 1234 };
const bar = foo;
console.log(bar.num); // 1234

bar.num = 0;
console.log(foo.num); // 0
```
上記の例では、変数`foo`と`bar`には同じオブジェクトが入っている。  
変数がそのオブジェクトの実体を専有しているとは限らず、他の場所で書き換えられる可能性がある。

下記の様に、明示的にオブジェクトをコピーすることで別のオブジェクトとして扱う事ができる。
1つめの方法がスプレッド構文を使用して次のように書く。  

```TypeScript
const foo = { num: 1234 };
const bar = { ...foo };
console.log(bar.num); // 1234
bar.num = 0;
console.log(foo.num); // 1234(書き換えられていない)
```

ただし、オブジェクトのプロパティの中に更にネストしてオブジェクトが入っている場合は同じオブジェクトのままなので注意が必要。

また、オブジェクトの比較には`===`を使用することができる。  
1つ目の例のように同じオブジェクトの場合はtrueとなり、２つ目の例のように中身が同じでも別々のオブジェクトの場合はfalseになる。

## まとめ

今回は名前と概要しか知らなかったTypeScriptに入門してみる記事でした。  
いざ勉強し始めるとその大部分はJavaScriptと同じことがわかってきて、よりハードルが下がった気がします。  
次回以降、TypeScriptならではの型について勉強していきたいと思います。  
また、実際にJavaScriptと比較してメリットを実感するために個人開発にも積極的に導入していこうと思います。  
