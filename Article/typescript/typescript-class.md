---
title: JavaScriptにおけるクラスとTypeScriptにおけるクラスについて
tags: [TypeScript]
createDate: 2023-01-08
updateDate: 2023-01-08
slug: typescript-class
---



## はじめに
今回はES2015から変更されたJavaScriptのオブジェクト指向構文についてまとめたいと思います。   
また、最近TypeScriptについて個人開発等で導入し始めているので違いを意識しながら理解していきたいと思います。

## 参考書籍
今回の記事は今読んでいるJavaScriptおよびTypeScriptの書籍を比較しながら書きました。   

[改訂新版JavaScript本格入門 ~モダンスタイルによる基礎から現場での応用まで](https://www.amazon.co.jp/%E6%94%B9%E8%A8%82%E6%96%B0%E7%89%88JavaScript%E6%9C%AC%E6%A0%BC%E5%85%A5%E9%96%80-%E3%83%A2%E3%83%80%E3%83%B3%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%81%AB%E3%82%88%E3%82%8B%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E7%8F%BE%E5%A0%B4%E3%81%A7%E3%81%AE%E5%BF%9C%E7%94%A8%E3%81%BE%E3%81%A7-%E5%B1%B1%E7%94%B0-%E7%A5%A5%E5%AF%9B/dp/477418411X)   
[プロを目指す人のためのTypeScript入門 安全なコードの書き方から高度な型の使い方まで](https://www.amazon.co.jp/%E3%83%97%E3%83%AD%E3%82%92%E7%9B%AE%E6%8C%87%E3%81%99%E4%BA%BA%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AETypeScript%E5%85%A5%E9%96%80-%E5%AE%89%E5%85%A8%E3%81%AA%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E6%9B%B8%E3%81%8D%E6%96%B9%E3%81%8B%E3%82%89%E9%AB%98%E5%BA%A6%E3%81%AA%E5%9E%8B%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9%E3%81%BE%E3%81%A7-Software-Design-plus/dp/4297127474)

## JavaScriptにおけるクラス構文

```JavaScript
class Member {
    constructor(firstName, lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // メソッド
    getName() {
        return this.lastName + this.firstName;
    }
}

let m = new Member('太郎', '山田');
console.log(m.getName());
```

### class命令

他のオブジェクト言語に近い形でclass命令を書くことができます。
```JavaScript
class クラス名 {
    // コンストラクターの定義
    // プロパティの定義
    // メソッドの定義
    メソッド名(引数) {
        メソッドのロジック
    }
}
```
コンストラクターの名前は`constructor`で固定です。

他言語との違いとして、`public/protected/private`のようなアクセス修飾子は利用できない点に注意が必要です。
JavaScriptにおいてはクラスのすべてのメンバーがpublic、つまりどこからでもアクセスできるようになります。


### 無名クラス(匿名クラス)について

少し特殊な書き方で、無名クラスと呼ばれる書き方ができます。   
リテラルなので、関数リテラルと同じく、式の中で利用できます。   

```JavaScript
let Member = class {
    constructor(firstName, lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
    getName() {
        this.lastName + this.firstName;
    }
}

let m = new Member('太郎', '山田');
console.log(m.getName());
```

また、無名クラスを定義して即時newすることもできます。   
クラスのインスタンスが生成されるため、変数に代入することも可能です。   

```JavaScript
new class {
    constructor(name) {
        console.log('hi!', name);
    }
}
('Yamada');
```

使いみちについてはぱっと思いつきませんが、こういう書き方ができると知っておくだけでいつか役に立つかも、、、？

### class命令は内部的には関数である

class命令で定義されたクラスは内部的に「特別な関数」として扱われます。   
つまり、ES2015にていわゆる`クラス`が導入されたわけではなくあくまで「これまでFunctionオブジェクトで表現していたクラス(コンストラクター)」をよりわかりやすく表現できるようになったに過ぎません。   
ただし、class命令によって定義されたクラスはFunctionオブジェクトによるクラスと完全には等価ではなく、どの部分で違いがあるのかについてこれから見ていきます。   

1. 関数としての呼び出しはできない   

例えばclass命令で定義されたMemberクラスを以下のように呼び出すことができません。
```JavaScript
let m = Member('太郎', '山田');
```

functionでのクラスの表現の場合は呼び出せてしまうため、以下のように対策する必要がありました。

```JavaScript
let Member = function(firstName, lastName) {
    if(!(this instanceOf Member)) {
        return new Member(firstName, lastName);
    }
    this.firstName = firstName;
    // 以下略
};
```
上記は、コンストラクターが関数として呼び出された場合にthisが`Member`オブジェクトではなくグローバルオブジェクトになる性質を利用してthisが`Member`オブジェクト出ない場合に改めてnew演算子でコンストラクターを呼び出しています。   

2. 定義前のクラスを呼び出すことはできない

以下のようなコードを書くことはできません。(function命令の場合は呼び出せます)
```JavaScript
let m = new Member('太郎', '山田');
// Member is not defined.
class Member {...中略...}
```

### class命令によるプロパティの定義

classブロックにおいて、get/set構文を使ってプロパティを定義することもできます。
```JavaScript
class Member {
    constructor(firstName, lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
    // プロパティの定義
    get firstName() {
        return this._firstName;
    }

    set firstName(value) {
        this._firstName = value;
    }

    get lastName() {
        return this._lastName;
    }

    set lastName(value) {
        this._lastName = value;
    }
    // メソッドの定義
    getName() {
        return this.last.Name + this.firstName;
    }
}

let m = new Member('太郎', '山田');
console.log(m.getName()); // 山田太郎
```

他の言語のプロパティ定義のように`let firstName = 'hogehoge'`のような書き方はできませんが、直感的なプロパティの定義だと思います。

### class命令によるその他のオブジェクト指向操作

1. 静的メソッドの定義   
`static`をメソッド定義の頭に付与することで静的メソッドを定義することができます。

2. クラスの継承   
`extends`を利用することで既存クラスを継承したサブクラスをシンプルに定義することができます。

```JavaScript
class Member {
    ...中略...
}

class BusinessMember extends Member {
    work() {
        return this.getName() + 'は働いています。';
    }
}

let bm = new businessMember('太郎', '山田');
console.log(bm.getName()); // 山田太郎
console.log(bm.work()); // 山田太郎は働いています。
```

3. オーバーライドと`super`キーワード   
基底クラスで定義されたメソッド/コンストラクターは、サブクラスで上書きすることもできます。
これをメソッドのオーバーライドと呼び、JavaScriptにおいてもこの仕組みが用意されています。

```JavaScript
class Member {
    ...中略
}
class BusinessMember extends Member {
    // Memberのコンストラクタに役職を追加
    constructor(firstName, lastName, position) {
        super(firstName, lastName);
        this.position = position;
    }
    getName() {
        return super.getName() + '/役職: ' + this.position;
    }
}
let bm = new BusinessMember('太郎', '山田', '課長');
console.log(bm.getName()); // 結果: 山田太郎/役職: 課長
```
オーバーライドは、基底クラスの機能を完全に書き換えるより、差分の処理を追加する場合が多いかなと思います。   
そのような場合に`super`を使用することで基底クラスの処理を利用しつつ新しいクラス定義が可能になります。   

## TypeScriptにおけるクラス構文

ここまで、JavaScriptにおけるクラスについて復習も兼ねて丁寧に確認してきました。   
ここからはクラス構文をTypeScriptでどのように書くのかについてまとめてみたいと思います。   
基本的な使い方にあまり差がない部分については省略して、JavaScriptと比較して有用な箇所や使用に注意が必要な箇所についてまとめます。

### プロパティの宣言

TypeScriptでは、JavaScriptでは不要だったプロパティの宣言を行う必要があります。   
定義していないプロパティアクセスはエラーとなります。   
以下の例のように、`プロパティ名: 型 = 式;`のかたちで書く事ができます。

また、下記の例は初期値を書いていますが、省略することもできます。   
ただし、初期値を省略する場合はコンストラクタを必ず書く必要があります。


```TypeScript
class User {
    name: string = "";
    age: number = 0;
}
```

- オプショナルなプロパティや読み取り専用のプロパティ

以下のような形でオプショナルなプロパティ(任意のプロパティ)や読み取り専用のプロパティを宣言することができます。

```TypeScript
class User {
    name?: string;  // オプショナルなプロパティ
    readonly age: int = 0;  // 読み取り専用プロパティ
}
const u = new User();
console.log(u.name); // undefinedが表示される(エラーではない)
console.log(u.age); // 0が表示される
u.age = 2; // エラー
```

読み取り専用プロパティは基本的に代入不可能ですが、コンストラクタの中では代入が可能です。

### 3種類のアクセス修飾子

TypeScriptでは、`public`, `protected`, `private`の３種類のアクセス修飾子をクラス宣言内のプロパティ宣言、及びメソッド宣言に付与することができます。   

- public ... どこからでもアクセス可能
- private ... クラスの内部からのみアクセス可能
- protected ... そのクラス自身と子クラスからアクセス可能

省略した場合は`public`と同じになります。   
これによって、`private`が付与されたプロパティやメソッドは外向きのインターフェースと内部実装とにはっきりと区分されます。   

### コンストラクタ引数でのプロパティ宣言

アクセス修飾子を用いることで、コンストラクタの引数でのプロパティ宣言が可能になります。
```TypeScript
class User {
    name: string;
    private age: number;

    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
```
上記のコードをコンストラクタ引数でプロパティ宣言を行うと以下のようになります。

```TypeScript
class User {
    constructor(public name: string, private age: number){}
}
```
コンストラクタの引数名の前にpublic,privateというアクセス修飾子が付きます。   
これによってコンストラクタ引数であると同時にプロパティ宣言であるとみなされます。   
ただし、この書き方をする場合はpublicの場合でもかならず修飾子が必要になります。   

処理は短くなりますが、プロパティ宣言を1箇所にまとめる事ができない点と、JavaScript本来の構文からかなり逸脱しているためこの書き方は好みが分かれますが、こういう書き方ができるということは知っておくといいと思います。

## まとめ

- JavaScriptにおけるclass構文はES2015で追加された比較的新しいもの
- プロパティの宣言にはコンストラクタを利用したものとget/setを利用したものがある
- TypeScriptにおけるclass構文ではプロパティの宣言が必要
- TypeScriptではプロパティとメソッドにアクセス修飾子が付与できる

## 参考

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes   
https://future-architect.github.io/typescript-guide/class.html   

## 所感

今回はJavaScriptのclass構文とTypeScriptのclass構文の違いを主に学習しました。   
TypeScriptでは他にも型引数やクラスの型の概念があるので引き続き調べていく必要がありそうです。   

2023年になり、年末は帰省していたため、久々に業務以外で一切勉強しない日が続きました。   
気分転換あまりできていなかったのでいい休息になったかなと思います。   
新年になったのでまた新たに目標を設定して2023年も駆け抜けたいと思います。