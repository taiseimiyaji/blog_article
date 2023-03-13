---
title: Factoryパターンについて
tags: [Design]
createDate: 2022-06-02
updateDate: 2022-06-02
slug: factory
---

先日チーム内でFactoryパターンについて話し合う機会があったのでその内容をまとめたいと思います。   
個人的にFactoryパターンについてよくわかっていなかったので質問する形でいろいろメンバー間で話し合ってみました。

## 参考
今回のFactoryパターンについては以下のものを参考にしました。

以下通称デザインパターン本   
GoFのデザインパターン本と呼ばれる原典

https://www.amazon.co.jp/%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B%E5%86%8D%E5%88%A9%E7%94%A8%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3-%E3%82%A8%E3%83%AA%E3%83%83%E3%82%AF-%E3%82%AC%E3%83%B3%E3%83%9E/dp/4797311126

結城浩さんのJavaで書かれたデザインパターン本(持ってる)

https://www.amazon.co.jp/%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82%E7%89%88Java%E8%A8%80%E8%AA%9E%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E5%85%A5%E9%96%80-%E7%B5%90%E5%9F%8E-%E6%B5%A9/dp/4797327030/ref=tmm_other_meta_binding_swatch_0?_encoding=UTF8&qid=&sr=

Factoryパターンについてのqiita記事

https://qiita.com/shoheiyokoyama/items/d752834a6a2e208b90ca

## 何がわからないのか

- なぜFactoryパターンを使うのか？という目的がピンときてない。

- 実際に携わっているプロダクトでもFactoryパターンを使っていて、見よう見まねで実装はできるが、この機会に目的をきちんと押さえておきたい。

## 自身で調べたこと

### Factory Methodパターン

>インスタンス作成をサブクラスに任せる

>スーパークラスでインスタンスの作り方の骨組みを定め、具体的な作成そのものはサブクラスで行う

### サンプル
結城浩さんの本では以下のサンプルが紹介されている

身分証明書カード(IDカード)を作る工場を題材としたもの。
本はJavaで書かれているのでPHPで書き直してみました。


ちなみにJavaの場合はパッケージが具体と抽象で分かれる。(PHPでいうnamespaceに近いかも)

パッケージについての参考記事

https://kanda-it-school-kensyu.com/java-basic-contents/jb_ch07/jb_0702/#:~:text=%E3%80%8C%E3%83%91%E3%83%83%E3%82%B1%E3%83%BC%E3%82%B8%E3%80%8D%E3%81%A8%E3%81%AF%E3%80%81Java,%E3%81%A6%E3%81%84%E3%81%8F%E5%A0%B4%E5%90%88%E3%81%8C%E3%81%82%E3%82%8A%E3%81%BE%E3%81%99%E3%80%82

抽象

- Factory
- Product


具体

- IDCardFactory
- IDCard

Factory.php
```php
<?php
declare(strict_types=1);

abstract class Factory
{
    public function create(string $owner): Product
    {
        $product = $this->createProduct($owner);
        $this->registerProduct($product);
        return $product;
    }

    abstract protected function createProduct(string $owner): Product;

    abstract protected function registerProduct(Product $product): void;
}
```

Product.php
```php
<?php
declare(strict_types=1);

abstract class Product
{
    abstract function use(): void;
}
```

IDCardFactory.php
```php
<?php
declare(strict_types=1);
require('./Factory.php');
require('./IDCard.php');

class IDCardFactory extends Factory
{
    private array $owners;

    public function __construct()
    {
        $this->owners = [];
    }

    protected function createProduct(string $owner): Product
    {
        return new IDCard($owner);
    }

    protected function registerProduct(Product $product): void
    {
        $this->owners[] = $product->getOwner();
        return;
    }
    public function getOwners(): array
    {
        return $this->owners;
    }
}
```

IDCard.php
```php
<?php
declare(strict_types=1);
require('./Product.php');

class IDCard extends Product
{
    private string $owner;

    public function __construct(string $owner)
    {
        echo $owner . "のカードを作ります。" . PHP_EOL;
        $this->owner = $owner;
    }
    public function use(): void
    {
        echo $this->owner . "のカードを使います。" . PHP_EOL;
    }
    public function getOwner(): string
    {
        return $this->owner;
    }
}

```

JavaのコードをPHPで書いているので少し不自然ですがMainクラスを用意しました。

main.php
```php
<?php
declare(strict_types=1);
require('./IDCardFactory.php');

class Main{
    public function __construct()
    {
        $factory = new IDCardFactory();
        $card1 = $factory->create('結城浩');
        $card2 = $factory->create('とむら');
        $card3 = $factory->create('佐藤花子');
        $card1->use();
        $card2->use();
        $card3->use();
    }
}

new Main();

```
## 特徴
- コンストラクタを直接呼ばずにインスタンスを生成できる。
- コンストラクタの生成時にValueObjectを引数として使う場合にnewする箇所を凝集できる
- クラスの生成に手順とかがあったらそのロジックを封じ込められる
- 多態性を利用してFactory内で分岐ロジックを書ける

以下のサイトには最大のメリットは「依存性の逆転」が実現できること、と書いてあるがピンときてない。

https://blog.ecbeing.tech/entry/2021/01/20/114000

----------------------------------------------------------------------------------------
ここから実際にメンバー間で話して得た知見を記述します。

Factoryパターンの目的

- Factoryパターンを使わずにUsecaseにnewする処理を書くということは具体に依存してしまっているということ。
- Repositoryパターンはデータベースに関連する処理を押し付ける。
- それと同じ感じでインスタンスの作成に関する処理をFactoryに押し付ける

- 工場のラインを想像すると良い   
->僕は工場の例えをものを作成する場所と考えてたけどライン工場を想像すると良いかも。   
必要な部品(VO)を用意したり、自明な値を持ってきたり、最低限必要なものを引数で受け取ったりしてものを作る工程をFactoryが担う。



## 「依存性の逆転」がわからん
ということで解説してもらったことをメモしておきます。

https://zenn.dev/naas/articles/c743a3d046fa78

上記ブログを参考に話を進める。

AとBの間に`IB`というインターフェースを作成することで依存性を逆転させることができる。

1. AがBに依存している

から

2. `AとIB`にBが依存する

に逆転させることができる。

Aを抽象に依存させることができる。

->Aの中で記述しているのはinterfaceの型とAPI(メソッド名)のみ。



Aがインターフェースを提供(IB)し、Bがインターフェースを実装してAに注入する、というパターンではAは一切Bに依存しません。   
つまりAの中では具体的なクラスは登場せず、インターフェースのみを使用します。   
Bを利用するためにはBに依存しないといけないところをAはBに依存せずに利用し、逆にBがAのインターフェース(IB)を実装するためにAに依存するのが「逆転」です。   
ここでのポイントはBが実装している**インターフェース(IB)はAが提供している**、と考えることです（僕はここがわかっていなかった）。


## 依存性の注入について
LaravelではDI(依存性の注入)の実現のため、サービスコンテナやサービスプロバイダという仕組みを使用している。というだけ。   
そもそもクラス内で使用するオブジェクトのインスタンスをクラス外で生成することを**依存性の注入**と呼ぶ。   
依存性の逆転を起こすための手段が依存性の注入。(用語が似ていてややこしい)

## サンプルコードを見直してみる
**依存性の逆転**についての理解ができたところでもう一度サンプルコードを見てみましょう。   
今回のUseCaseに当たるMainクラス内ではIDCardをnewすることなくIDCardを使用しています。   
つまりIDCardへの依存をせずにインターフェースへの依存(createメソッドやuseメソッドを使用できることを期待しているため)をするように変更されました。   
Factoryの中でどんなクラスが生成された場合でも、それがProductインターフェースさえ実装していればMain(UseCase)では意識することなく使用することができます。   
Factoryパターンを用いることで**依存性の逆転**を起こすことができています。   
また、現状ではIDCardFactoryをnewしているため、具体的なクラスに依存していますが、これもDIすることでさらに依存関係を改善できそうです。   
ここまでいろんな手法を用いて依存関係を整理してきました。どうして依存性にこんなに気を遣ったつくりにするのでしょうか。   
大きなメリットとして以下の二つがあるのかなと考えています。

- 外部を意識せずに対象クラスのテストを書きやすくなる
- 特定のフレームワークに依存しなくて済む

まずは一つ目について。   
インターフェースに依存する作りにすることで、インターフェースさえ実装していれば使用することができるようになります。   
これを利用してテストしたいクラスに必要なものをMockに置き換えることができるようになります。   
よりテスタブルなコードが実現できます。   

二つ目について   
インターフェースに依存させ、具体的な実装を引き剥がすことで特定のフレームワークに依存する処理を封じ込めることができるようになります。   
例えばLaravelの場合、DBの操作には`Eloquent`という仕組みを使用することが多いです。   
では途中でLaravelから他のフレームワークへ移行することを考えてみましょう。   
今回のFactoryパターンと同じく依存性の逆転を実現するためのパターンとして、Repositoryパターンというものがあります。   
このパターンを用いることでLaravelに依存しているコードをRepositoryに封じ込めることができます。   
つまり、Repositoryさえ修正すればUseCaseのロジックを修正することなくフレームワークの移行が完了します。   

上記二つのメリットはいずれも**依存性の逆転**を起こすことで変更容易性を向上させ、それに伴って発生するメリットです。

## まとめ
- インスタンス生成に関わるロジックを担うのがFactory
- その利点を理解するには依存性の逆転や依存性の注入について理解する必要があった
- 有効なパターンを使用することで変更容易性を向上させることができ、テスタブルなコードになる。


## 所感
**依存性の逆転**を理解することでFactoryパターンのメリット、というか世の設計手法やデザインパターンの目指すところが掴めてきた感じがする。　　　
個人的にはアンチパターンを踏み抜く経験がそもそもないからパターンのメリットが実感できてないのかもしれない。   
車輪の再発明を積極的にしていくべきなのか、どういう勉強をしていくべきなのかが難しい。   
クリーンアーキテクチャを具体的にどう実現していくのかわかってきた気がする。   

