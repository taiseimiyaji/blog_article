---
title: TypeScriptの型について
tags: [TypeScript]
createDate: 2023-01-29
updateDate: 2023-01-29
slug: typescript-type
---

## TypeScriptの型付け

TypeScriptはJavaScriptに対して型を付与するという思想で仕様が定められています。   
TypeScriptでは型を付与する方法として、様々な方法が用意されていますが、どこまで利用するかは費用対効果を考えながら行う必要があります。   

## any型 最もゆるい型付け

```TypeScript
function example(args: any){
    // argsにhogeが存在するかのチェックはしないのでコンパイルエラーとはならない
    console.log(args.hoge);
}
```

any型を使った場合、TypeScriptの型チェックの恩恵を受けることができません。  
any型は型チェックを無効化する型です。any型の変数になにかを代入することや、any型の値を他の型の変数に代入することに対してもコンパイルエラーは発生しません。  
JavaScriptからの移行時等に一時的に利用するなど以外は原則使用しないようにするべきです。  


## unknown型

unknown型はany型と似ています。   
「型安全なany型」と呼ばれ、`何でも入れられる型`です。  
unknown型にはどのようなデータもチェックなしに入れる事ができます。   
any型と違う点は変数を利用する場合に型アサーションを使ってチェックを行わないとエラーになる点です。  

any型はどのような型の変数にも代入できますが、unknown型の値は具体的な型へ代入できません。

```TypeScript
const value: any = 10;
const int: number = value;
const bool: boolean = value;
```


また、ジェネリクスを使ったクラスや関数のうち、自動で型推論で設定できなかったものが`unknown`となります。  
この型変数の`unknown`に関してはエラーチェックなどが行われることがなく、`any`のように振る舞います。  

unknownの用途としてはany型の値をより安全に扱うことです。`unknown`に対して許可される操作は限定的です。  
例えば、ピリオドを使用してメンバーアクセスをしたり、メソッド呼び出しをしようとするとコンパイルエラーとなります。  
any型を扱う場合は一旦unknown型にしておくことで存在しないプロパティへのアクセスにコンパイル時に気づきやすくなります。  

- unknownと型の絞り込み

unknownの値を実用的に使うためには型を絞り込む必要があります。  
型の絞り込みには`typeof`や`instanceof`などを条件式に含んだif文やswitch文を使います。
これは**型ガード**といい、それ以降の処理では絞り込まれた型として扱う事ができます。

```TypeScript
function useUnknown(value: unknown) {
    if (typeof value === "string") {
        // この時点でvalはstring型になる
        console.log(value.toUpperCase());
    }
}
```


## ユニオン型とインターセクション型

日本語でいう「または」や「かつ」を表現する型です。

- ユニオン型

「型Tまたは型U」のような表現ができる型です。  
「T | U」のように書きます。

```TypeScript
type Animal = {
    species: string;
}

type Human = {
    name: string;
}

type User = Animal | Human;

const tama: User = {
    species: "calico"
}

const you: User = {
    name: "Your Name"
}

```

「ユーザーには動物と人間の2種類がある」という場合、つまりユーザーは動物または人間である」という場合を想定しています。   
下記のように、コンパイラによる型チェックを受けられます。
```TypeScript
// エラーが発生する
const book: User = {
    title: "Software Design"
};
```
User型を持つことだけがわかっている場合は実際にはそれがAnimalなのかHumanなのか不明です。

```TypeScript
function getName(user: User): string {
    return user.name;
}
```
userはAnimalかもしれないしHumanかもしれません。Animalにはnameというプロパティが存在しないため、userがnameを持たない可能性がある場合はコンパイルエラーになります。   
つまり、上記で作成したUser型は全くプロパティアクセスができません。  

反対に、必ず存在するプロパティの場合は下記のようになります。
```TypeScript
type Animal = {
    species: string;
    age: string;
}

type Human = {
    name: string;
    age: number;
}

type User = Animal | Human;

const tama: User = {
    species: "calico",
    age: "永遠の17歳"
}

const you: User = {
    name: "Your Name",
    age: 25
}

function showAge(user: User) {
    const age = user.age;
    console.log(age); // ageは string | number型となる
}
```
ユニオン型に対するプロパティアクセスが可能である場合、その結果はユニオンの構成要素それぞれのプロパティの型を集めたユニオン型となります。  

- インターセクション型

「T & U」のように書く、「T型かつU型」を意味する型です。`交差型`とも呼ばれます。  
例えば、下記のように「HumanはAnimalの一種である」ことを表現します。

```TypeScript
type Animal = {
    species: string;
    age: number;
}

type Human = Animal & {
    name: string;
}

const tama: Animal = {
    species: "calico",
    age: 3
};

const you: Human = {
    species: "Homo sapiens",
    age: 26,
    name: "Your Name"
}
```
Human型は、Animal型を拡張してstring型のnameプロパティを持ちます。  
つまり、下記と同様になります。
```TypeScript
type Human = {
    species: string;
    age: number;
    name: string;
}
```
また、&で作られた型はそれぞれの構成要素の型の部分型となります。   
`HumanはAnimalの部分型`となります。  


## ユニオン型とインターセクション型の関係

```TypeScript
type Human = {name: string};

type Animal = {species: string};
function getName(human; Human) {
    return human.name;
}
function getSpecies(animal: Animal) {
    return animal.species;
}
const mysteryFunc = Math.random() < 0.5 ? getName : getSpecies;
```

変数mysteryFuncにはgetNameが入るかもしれないしgetSpeciesが入るかもしれません。  
この場合、mysteryFuncは以下の様な型になります。

```TypeScript
((human: Human) => string | ((animal: Animal) => string))
```

mysteryFuncを関数として呼び出したい場合、Human型を受け取るとは限らないのでHumanを渡すことはできないし、その一方でAnimalを受け取るとは限らないのでAnimalを渡すこともできません。  
ユニオン型を持つ関数はどんな引数を受け取るのか不明なので扱いが困難です。   

ここでmysteryFuncを呼び出す方法は、Human & Animal型を渡すことです。  
つまり、ユニオン型とインターセクション型は全くの無関係ではなく、ユニオン型からインターセクション型が生み出される場合もあります。   
「AND」と「OR」は論理学的にも表裏一体の関係なので不自然ではないです。   


異なるプリミティブ型同士でインターセクション型を作った場合は`never型`が出現します。

## never型

`unknown`型の真逆の存在で、「当てはまる値が存在しない」という性質を持ちます。  
`never`型にはnever型以外何も代入できません。  
正規の手段でnever型の値を得ることは不可能であり、言い換えるとnever型の値が存在しているコードは実際には実行されません。  
ただし、never型はどんな型にも代入することができます。  

```TypeScript
const nev = 1 as never;
const a: string = nev;
const b: number = nev;
const c: string[] = nev;
```

これは、「never型はすべての型の部分型である」からです。  
また、never型はユニオン型の中では消えることも把握しておく必要があります。  
例えば、string | never はstringと同じです。  


## ユーザー定義型ガード(user-defined type guards)

ユーザー定義型ガードとは、型の絞り込みを自由に行うためのしくみです。  
注意点として、ユーザー定義型ガードはanyやasの仲間であり、型安全性を破壊する恐れのある危険な機能の一つです。  
`型述語(type predicates)`と呼ばれるものを返り値の型に書きます。  
型述語の書き方には以下の2種類があります。  


- 引数名 is 型
- asserts 引数名 is 型

```TypeScript
function isStringOrNumber(value: unknown): value is string | number {
    return typeof value === "string" || typeof value === "number";
}

const something: unknown = 123;

if(isStringOrNumber(something)) {
    //この時点でsomethingは string | number型
    console.log(something.toString());
}
```

下記のように、`isStringOrNumber`の返り値をbooleanに変えてみると、コンパイルエラーとなります。

```TypeScript
function isStringOrNumber(value: unknown): boolean {
    return typeof value === "string" || typeof value === "number";
}

const something: unknown = 123;

if(isStringOrNumber(something)) {
    // エラー: Object is of type 'unknown'.
    console.log(something.toString());
}
```

ユーザー定義型ガードは関数を絞り込みに使うことができます。ですが、その関数の型定義でユーザー定義型ガードを使わなければいけません。  
注意点としては、ユーザー定義型ガードは関数の実装内容はTypeScriptの保証する範囲ではないことです。  
下記のような間違った実装をしてしまった場合でもコンパイルエラーは発生しないため、型安全性を破壊することになります。  

```TypeScript
function isStringOrNumber(value: unknown): value is string | number {
    // 実装を間違えているがエラーが起きない!
    return typeof value === "string" || typeof value === "boolean";
}
```


## 所感

今回はTypeScriptの型の一部について調査しました。  
特にユニオン型とインターセクション型は使用頻度が高くなりそうな型でした。  
TypeScriptには他にも高度な型が用意されていますが、今回の記事である程度基本的な型をおさえることができたので、残りについてはなにか作ってみてから調査したいと思います。  
個人的には最近のフロントエンド界隈ではastroが気になっているので入門記事も今度書いてみようと思います。  

## 参考

https://future-architect.github.io/typescript-guide/typing.html  
https://typescriptbook.jp/
　
