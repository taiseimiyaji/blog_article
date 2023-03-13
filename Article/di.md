---
title: プログラミングにおける「DI」二種類の違いとは。依存性注入と依存性逆転について
tags: [Laravel]
createDate: 2022-12-06
updateDate: 2022-12-06
slug: di
---
# 依存性注入と依存性逆転について

## はじめに
エンジニアになって初めて人に知識を伝えるという場面がありました。  
これまでチーム内ではたいてい自分が一番技術的に拙い部分が多かったので少しうれしくもありますが、正しくわかりやすく伝えるために勉強してきたことを棚卸しするとともに現時点での考えを整理していきたいと思います。

## DIとは

プログラミングの文脈で登場する**DI**という単語には以下の二種類があります。

- 依存性逆転の原則(Dependency inversion principle)
- 依存性注入(Dependency Injection)

混乱しやすいのは両者はかなり密接な関係にあります。
この２つのDIを理解するために、まずは依存性逆転の原則について理解を進めます。

## 依存性逆転の原則

**依存性逆転の原則**とは、一言でいうと「抽象へ依存させることで依存関係を逆転させる」という原則です。
次の２つが中心的な考え方になります。

1. 上位のモジュールは下位のモジュールに依存してはならない。双方とも抽象（例としてインターフェース）に依存するべきである。
2. 「抽象」は実装の詳細に依存してはならない。実装の詳細が「抽象」に依存すべきである。

理解を進めるにあたって、「依存」について知っておく必要があります。  
JSでいうとimport、PHPでいうとuseなどを使用してモジュールを使う側が依存する側です。  
反対に、使用される側が依存される側です。
依存先のモジュールがないとモジュールが成り立たない状態を依存先のモジュールに「依存している」といいます。

プログラミングにおいては、例えば重要性が高いビジネスロジックをまとめたドメイン層の部分と表示における関心事をまとめたプレゼンテーション層では変更頻度が異なります。

それぞれの層に求められる性質は以下のようになります。

ドメイン層->安定性が高い、柔軟性は求められない  
プレゼンテーション層->安定性が低い、柔軟性が求められる  

システムの本質であるドメイン知識(ドメイン層)は最も安定性が高くなければなりません。  
モジュール間に依存関係が成立する場合、依存する側は安定性が低く、柔軟性が高くなります。  
これはモジュールを使用すればするほどコードが複雑になるからです。  

今回はPHPを例に依存性逆転の原則を適用した場合とそうでない場合を比較してみます。  
前回の記事にて軽量DDDを紹介しているのでそれに合わせたレイヤ構造を例にしてみます。

- リポジトリ層
- サービス(ドメイン)層

### 従来のレイヤーパターン

[依存性逆転の原則のwikipedia](https://ja.wikipedia.org/wiki/%E4%BE%9D%E5%AD%98%E6%80%A7%E9%80%86%E8%BB%A2%E3%81%AE%E5%8E%9F%E5%89%87)に下記の説明が記載されています。過不足なくわかりやすい説明だと思ったので引用しておきます。

>伝統的なアプリケーションアーキテクチャにおいては下位レベルのコンポーネントはより複雑なシステムの構築を可能にする上位レベルコンポーネントによって使用される形で設計がおこなわれる。この方法では上位レベルコンポーネントは直接下位レベルコンポーネントに依存する。この低レベルコンポーネントへの依存は、上位レベルコンポーネントの再利用の機会を制限してしまう。

実際にPHPで書いてみると以下のようなコードになります。C言語のような手続き型言語の場合は結構使用されているパターンだと思います。簡単に言うと部品を作ってそれをまとめ上げるイメージです。

(イメージなので実際には動作しないと思いますので注意してください)
```PHP
class Service
{
    private Repository $repository;

    public function __construct()
    {
        $this->repository = new Repository();
    }

    public function process()
    {
        return $this->repository->findAll();
    }
}

class Repository
{
    public function findAll()
    {
        // 具体的なDBアクセス処理
        $result = 'hoge';
        return $result;
    }
}

```

このように上位のレイヤから下位のレイヤを呼び出す形がレイヤーパターンと呼ばれていたりします。
注目してほしいのはコンストラクタで直接newをしている部分です。このように直接newをしている場合はRepositoryという具象クラスの実装に依存してしまいます。

この場合、システムの本質であるServiceがRepositoryに依存しているため安定性が低くなってしまいます。  
ここでいう`安定性が低い`というのは他のモジュールの影響を受けやすく、変更に弱いことを意味しています。  
安定性を上げるために、ドメイン層であるServiceはモジュールをuse(具象クラスをnew)しないように書くべきです。  

## 依存性注入

さて次はどうやってドメイン層の安定性を上げていくのかについて考えてみます。  
ここで登場するのが依存性注入です。  
**依存性注入**とはプログラミングにおける設計思想の一種です。  
コードの実行時にインターフェース(または抽象クラス)に対して具象クラスを外部から注入(Inject)するという考え方です。  
オブジェクトが他のオブジェクトを利用する際、コードをはじめから結合させるのではなく、オブジェクトの実行時に呼び出して結合するようにしています。

広義的には対象モジュール以外のモジュールから具象クラスが渡される場合はDIと呼びます。

先程のレイヤーパターンの例でDIを使用してみます。

```PHP
use Repository;
class Service
{
    private Repository $repository;

    public function __construct(Repository $repository)
    {
        $this->repository = $repository;
    }

    public function process()
    {
        return $this->repository->findAll();
    }
}
```
```PHP
class Repository
{
    public function findAll()
    {
        // 具体的なDBアクセス処理
        $result = 'hoge';
        return $result;
    }
}

// ここで依存性を注入
$service = new Service(new Repository());
$action = new Action($service);
// 実行
$action->process();
```

変更点はコンストラクタにて外部からインスタンスを渡すように変更している点です。  
外部から具象クラスを渡す事によってクラス単位での単体テストが可能になります。  
どのクラスを使用するかは外部から決める事ができるので、モック等を使用しやすくなります。  
ここでServiceのコンストラクタにはRepository型のインスタンスを渡す必要がありますが、コンストラクタで渡すように変更したため、Repositoryクラスを継承したクラスを渡すことが可能になリます。

次のステップとして、Repositoryにインターフェースを実装します。

```PHP
use RepositoryInterface;
class Service
{
    private RepositoryInterface $repository;

    public function __construct(RepositoryInterface $repository)
    {
        $this->repository = $repository;
    }

    public function process()
    {
        return $this->repository->findAll();
    }
}

interface RepositoryInterface
{
    public function findAll();
}
```
```PHP
class Repository implements RepositoryInterface
{
    public function findAll()
    {
        // 具体的なDBアクセス処理
        $result = 'hoge';
        return $result;
    }
}

// ここで依存性を注入
$service = new Service(new Repository());
$action = new Action($service);
// 実行
$action->process();
```

ここでのポイントはドメイン層側にインターフェースを持つことです。  
ドメイン層側というのはPHPの場合はnamespaceにおけるドメイン層を指しています。(Javaの場合はパッケージにあたるかなと思います)  
明示的にドメイン層とその他の層を分けておくことをおすすめします。  

これによってドメイン層はRepositoryInterfaceに依存し、Repositoryの具体的な実装に依存しなくなります。  
反対に、Repositoryがnamespace的にドメイン層にあるRepositoryInterfaceに依存します。  

最初のレイヤーパターンと比較すると依存性が逆転しています。これを**依存性逆転の原則**と呼びます。  
このパターンを採用することでドメイン層はRepositoryに依存しないため、変更があった場合でも影響を受けにくくなります。

また、依存性注入はフレームワークやライブラリでおこなってくれる場合が多く、`DIコンテナ`と呼ばれたりします。
Laravelにおいてはサービスコンテナ、サービスプロバイダを理解するとイメージが湧くかなと思います。

[サービスコンテナ](https://readouble.com/laravel/8.x/ja/container.html)  
[サービスプロバイダ](https://readouble.com/laravel/8.x/ja/providers.html)

## アーキテクチャについて

**依存性逆転の原則**を利用したアーキテクチャとして良く取り入れられているものにクリーンアーキテクチャやオニオンアーキテクチャ、ヘキサゴナルアーキテクチャ等があります。

これらのアーキテクチャはどれも本質的なビジネスロジックをまとめた層に向けて依存させるという依存の方向性を重視しています。依存の方向を内側(ドメイン層)へ向けるための具体的なレイヤ分けが異なるだけで、本質となる考え方は共通していると私は考えています。

どのアーキテクチャもレイヤ分けしたそれぞれの層の責務を明確にして依存関係を明確にするという目的が共通しています。

責務を明確にすることで振る舞いを適切に抽象化してインターフェースを作成します。そのインターフェースに対して依存することで共通の振る舞いを持つ別クラスに交換可能になります。  
別のオブジェクトが同じふるまいを持ち、異なるクラスを同じものとみなせる性質をオブジェクト指向の文脈においてはポリモーフィズムと呼びます。

これにより交換可能なコード、つまり単体テストのしやすいコードになります。  
私個人の考えですが、良い設計というのは変更容易性を高めるために必然的に交換可能なコードになります。  
そのため、良い設計になっているかどうかの指標としてテストがしやすいかどうかは一つのポイントだと思っています。  
テストがしにくいなと感じた場合は設計が悪い可能性が高い、といえます。

先程挙げた例のServiceをテストしてみます。  
このとき、必要なのはRepositoryInterfaceのみであり、Repositoryの具体的な実装は必要ないため、Mock化することができます。

```PHP
use RepositoryInterface;
class Service
{
    private RepositoryInterface $repository;

    public function __construct(RepositoryInterface $repository)
    {
        $this->repository = $repository;
    }

    public function process()
    {
        return $this->repository->findAll();
    }
}

interface RepositoryInterface
{
    public function findAll();
}

```
```PHP
class ServiceTest
{
    public function testProcess()
    {
        $repositoryMock = new RepositoryMock();
        $result = new Service($repositoryMock);
    }
}

class RepositoryMock implements RepositoryInterface
{
    public function findAll()
    {
        // テストのための値を返す
        return 'test';
    }
}
```

## まとめ

- DIには、`依存性の注入(Dependency Injection)`と`依存性逆転の原則(Dependency inversion principle)`の2種類がある。

- **依存性逆転の原則**とは、一言でいうと「抽象へ依存させることで依存関係を逆転させる」という原則。
1. 上位のモジュールは下位のモジュールに依存してはならない。双方とも抽象（例としてインターフェース）に依存するべきである。
2. 「抽象」は実装の詳細に依存してはならない。実装の詳細が「抽象」に依存すべきである。

- 依存性の注入とは、コードの実行時にインターフェース(または抽象クラス)に対して具象クラスを外部から注入(Inject)するという考え方です、
## 所感

今回はDIについてメンバーに説明する機会があったのでまとめてみました。  
自分でサンプルコードを書いてみたり、人に説明することで自分自身の理解もより高まるなと感じています。  
ここ最近は自分の知識を吐き出すことが多かったのですが、さまざまな技術ブログでアドベントカレンダーイベントが始まっているので毎日いろんな記事を参考にできて最近はインプットも増えて嬉しいです。  
またなにか新しい考えを身に着けられれば記事にしたいと思います。

## 参考
https://qiita.com/okazuki/items/a0f2fb0a63ca88340ff6  
https://zenn.dev/chida/articles/e46a66cd9d89d1