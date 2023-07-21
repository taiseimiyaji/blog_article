---
title: LaravelにおけるRepositoryについて再考してみる
tags: [Laravel, Repository]
createDate: 2023-07-21
updateDate: 2023-07-21
slug: laravel-repository
---

## はじめに

ここ最近、ツイッター上でLaravelにおいてのRepositoryの実装についての記事や意見をいくつか見かけました。  
それらの記事や意見の内容を整理しつつ、自分なりにLaravelにおけるRepositoryの実装について再考してみたいと思います。  

## Repositoryとは

Repositoryは「エリック・エヴァンスのドメイン駆動設計」において紹介されているモデル駆動におけるクラスの一つです。

概要としては、ドメインモデルを永続化する処理を抽象化するために存在します。`リポジトリ`は直訳で保管庫を意味します。

今回は最初にコード例を示した上で、Repositoryの実装について考えていきます。

参考: https://zenn.dev/kohii/articles/e4f325ed011db8

## LaravelでRepositoryを実装してみる

### 設計

今回は、ユーザーを管理する一連の処理を題材にRepositoryを使って実装してみます。

クラスは下記のように作成します
```
ユーザー(User)
- ID(Id)
- 名前(Name)
- メールアドレス(Email)
```

永続化処理をRepositoryで抽象化し、具象としては以下のようなクラスを作成します。
```
- UserMySQLRepository
- UserSQLiteRepository
```

### 実装

Userクラス

```php
<?php
declare(strict_types=1);

namespace user;

/**
 * ユーザーを表現するクラス.
 * DDDの文脈におけるエンティティ.
 */
class User
{
    private string $id;
    
    private string $name;
    
    private string $email;

    public function __construct(
        string $id,
        string $name,
        string $email
    ) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
    }

    public function getId(): string
    {
        return $this->id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }
}

```

今回はLaravelのORMであるEloquentを使って永続化処理を実装してみます。

Repositoryを表現するためのInterfaceを作成します。

このインターフェースを持つものはRepository、つまり永続化処理を抽象化したものであるということがわかります。

簡略化のために、今回はfindByIdとsaveのみを定義します。

UserRepositoryInterface
```php
<?php
declare(strict_types=1);

namespace user;

interface UserRepositoryInterface
{
    public function findById(int $id): User;

    public function save(User $user): void;
}
```


具体的なRepositoryの実装です。

UserMySqlRepositoryクラスとUserSqliteRepositoryクラスを作成します。

UserMySqlRepositoryクラス
```php
<?php
declare(strict_types=1);

namespace user;

class UserMySqlRepository implements UserRepositoryInterface
{
    protected \App\Models\User $UserModel;

    private string $connection;

    public function __construct(\App\Models\User $UserModel, $connection = 'mysql')
    {
        $this->UserModel = $UserModel;
        $this->connection = $connection;
    }

    public function findById(int $id): User
    {
        $data = $this->UserModel::on($this->connection)->find($id);
        return new User(
            $data->getAttribute('id'),
            $data->getAttribute('name'),
            $data->getAttribute('email')
        );
    }

    public function save(User $user): void
    {
        $this->UserModel::on($this->connection)->updateOrCreate(
            ['id' => $user->getId()],
            [
                'name' => $user->getName(),
                'email' => $user->getEmail(),
            ]
        );
    }
}
```

UserSqliteRepositoryクラス
```php
<?php
declare(strict_types=1);

namespace user;

class UserSqliteRepository implements UserRepositoryInterface
{
    protected \App\Models\User $UserModel;

    private string $connection;

    public function __construct(\App\Models\User $UserModel, $connection = 'sqlite')
    {
        $this->UserModel = $UserModel;
        $this->connection = $connection;
    }

    public function findById(int $id): User
    {
        $data = $this->UserModel::on($this->connection)->find($id);
        return new User(
            $data->getAttribute('id'),
            $data->getAttribute('name'),
            $data->getAttribute('email')
        );
    }

    public function save(User $user): void
    {
        $this->UserModel::on($this->connection)->updateOrCreate(
            ['id' => $user->getId()],
            [
                'name' => $user->getName(),
                'email' => $user->getEmail(),
            ]
        );
    }
}
```

Repositoryはアプリケーションサービスと呼ばれるクラスから呼び出されます。

アプリケーションの要求に応じてサービスクラスは分割することがベターだと考えているのでここでは2つに分割します。

コントローラーについては今回は省略しますが、サービス同様に1コントローラー1メソッドとなるよう分割することがベターだと考えています。

CreateUserServiceクラス

```php
<?php
declare(strict_types=1);

namespace user;

class CreateUserService
{
    private UserRepositoryInterface $userRepository;

    public function __construct(UserRepositoryInterface $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function process(User $user): void
    {
        $this->userRepository->save($user);
    }
}
```

GetUserServiceクラス
```php
<?php
declare(strict_types=1);

namespace user;

class GetUserService
{
    private UserRepositoryInterface $userRepository;

    public function __construct(UserRepositoryInterface $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function process(int $id): User
    {
        return $this->userRepository->findById($id);
    }
}
```

## Repositoryの定義を再確認

原著である「エリック・エヴァンスのドメイン駆動設計」から、Repositoryの意味するところを引用し、確認します。

 >集約ルート以外のオブジェクトにグローバルなアクセスを提供してしまうと、重要な区別が台無しになる。データベースクエリを自由に行うと、実はドメインオブジェクトと集約のカプセル化に違反しかねないのだ。技術的なインフラストラクチャやデータベースアクセスの仕組みを露呈させてしまうと、クライアントが複雑になり、モデル駆動設計が不明瞭になってしまう。

Eric Evans. エリック・エヴァンスのドメイン駆動設計 (Kindle の位置No.3526-3530). Kindle 版.

>実際のストレージや問い合わせの技術をカプセル化すること。実際に直接的なアクセスを必要とする集約ルートに対してのみ、リポジトリを提供すること。クライアントをモデルに集中させ、あらゆるオブジェクトの格納とアクセスをリポジトリに委譲すること

Eric Evans. エリック・エヴァンスのドメイン駆動設計 (Kindle の位置No.3561-3563). Kindle 版.

といった記載があります。このあたりから読み取れることは、あくまでRepositoryはDDDにおける集約の単位で永続化する際に抽象化のために用いられることがわかります。

## アンチパターンを考えてみる

## RepositoryにEloquent Modelを渡してしまう

今回記事を書くに当たってTwitterで散見されたアンチパターンに、以下のようにEloquent Modelを使用しているものがありました。

ここで引数に渡しているUserクラスはエンティティではなく、Eloquent ModelのUserであることに注意してください。

```php
<?php
declare(strict_types=1);

namespace user;

use App\Models\User;

class BadUserRepository
{
    public function save(User $user)
    {
        return $user->save();
    }

    // その他のメソッド...
}
```

このような実装は原著の意図するRepositoryとは異なります。

また、Eloquentという具体的なデータアクセス手段が外部に漏洩しているのでRepositoryである意味がありません。無駄なクラス生成とロジックの分散を行っているだけとなってしまいます。

どうしてこのような無駄なRepositoryを作成してしまうのかという原因を考えてみたんですが、そもそも「Repositoryパターン」という言葉が独り歩きしている印象を受けます。

先程確認したRepositoryの説明から、私は以下のように解釈しています。

1. モデル駆動という考え方から、ドメインモデルをクラスの形で表現する
2. ドメインモデルは、複数の概念をまとめた「集約」として扱われることもある
3. ドメインモデルを表現したクラスはEntityと呼ばれ、各パラメータはValueObjectパターンで表現されることもある
4. このドメインモデルの永続化を抽象化するためのクラスがRepositoryである
5. Repositoryは永続化や再構築を行うが、扱うのはすべて「集約」単位である。

今回のアンチパターンは、RepositoryのInterfaceの時点でEloquentを使用しており、集約ではない上に、ORMというデータアクセスに関する知識をRepository外部に漏洩している点が問題です。

### ではEloquent Modelの存在意義は?

Laravelを採用する上でEloquentというのは非常に大きなメリットを持っています。

RailsのActiveRecordと同様に、データベースのテーブルとオブジェクトをマッピングすることができます。

ただ、このパターンはDDDの思想とは少し異なります。レイヤードアーキテクチャやDDDを採用する上で、データアクセスの知識をカプセル化する必要があるため、Eloquentのメリットを最大限に活かすことは難しくなってきます。

個人的に、Eloquentのメリットが活用できるのは、以下のような場合だと思っています。

1. 開発効率を重視し、レイヤードアーキテクチャやDDDを採用しない場合
2. ドメインモデルが実際のDBのテーブルと全く同じ構成であり、「集約」について意識しない場合

プロダクトの成長に伴って、メンテナンス性を意識し始めると、Eloquentのメリットを享受することは難しくなってきます。

resourceと完全に一致する構成で単純なCRUDのみを行う場合は、Eloquentを採用することで開発効率を上げることができると思いますが、Railsと同様にControllerにすべてを記載し、レイヤー分けしないメリットのほうが大きいと思います。

### アンチパターンを踏みがちなケース

ここからは完全に個人的な妄想と想像です。(ここまでも)

このアンチパターンを踏む思考は以下のようなものが挙げられます。

- レガシーなシステムを改善しないといけないので巷で噂の「Repositoryパターン」を採用してみる
- DDD? レイヤードアーキテクチャ? そんなものは知らないけど、Eloquentを使っているから大丈夫だろう

そもそも、「Repositoryパターン」ってなんなの？というところから疑問に感じています。  
あくまでRepositoryはDDDという大きな文脈に登場する役割の一つであって、GoFのデザインパターンのような特定のデザインパターンではないので、単体で活用することのできないものだと思っています。  
大体ネットで見かける危ない匂いのする「Repository」は「Repositoryパターン」を銀の弾丸だと盲信していて、原著やDDDの文脈に触れていない記事が多いような気がします。  

しっかりとモデルについて考えてクラスにした上で、そのモデルを取り回す際にRepositoryを活用するというのが、私の理解です。

### Laravelのコードの改善案

Controllerに処理が全て詰め込まれているようなコードを改善するために、Repositoryから導入するのは良くないというのがここまでの主張ですが、どのような段階を踏んで改善していくのが良いのかを考えてみます。

第一段階は「UseCase」だけを取り入れることです。

参考: https://zenn.dev/mpyw/articles/ce7d09eb6d8117

この段階では、Eloquentを積極的に活用するためにあえて分割する粒度を大きくしています。

第二段階はEloquentを捨て、DDDとレイヤードアーキテクチャにガッツリ寄せることです。

第一段階から飛躍していますが、やむを得ません。Laravelのお作法自体がレイヤードアーキテクチャやDDDといった設計手法とは異なっているので、そのままではなかなか改善できません。

この段階ではEloquent Modelを捨て、ファサードやヘルパーも捨てましょう。

参考: https://blog.ytake.jp.net/entry/2015/12/16/011812

要件が複雑なプロジェクト?寿命が長いプロダクト?そんな場合は覚悟を決めて第二段階を採用しましょう。

Eloquentを採用できないの?じゃあ、Laravelを使わないほうが良いのでは？という話になってしまいますが、採用されている数の多さやそれに伴うエンジニアの多さなどから選定されることも多いと思います。今回は既存プロジェクトの改善の話なのでLaravelプロジェクトを想定しています。

~~立ち上げ段階でみんなちゃんと設計できて扱えるならSymfony + Doctrineでいいんじゃ~~


## 【余談】 RepositoryにEloquent ModelをDIするかどうか

参考: https://qiita.com/fuwasegu/items/f778a1067c941dc8baea

RepositoryにEloquentModelをDIするかどうかについては、いくつかの意見があります。

個人的には、Repositoryの中にカプセル化できるのであれば、EloquentModelの採用自体には問題はないと思っています。

ただし、参考記事にあるようにActiveRecordの思想のメリットを十分に享受できるとは言えず、たとえばテーブル名や接続といった一部の情報を抽象化するためにEloquentを使用することになります。

多少もったいない感じはしますが、ここまで述べたDDDの各種パターンを守った上で、Eloquentを採用することが設計上大きな障害に直結することは無いと思っています。


## 所感

いいとされている設計手法には、きちんと目的や根拠があり、一種ルールがあります。

この「ルール」は、例えばレイヤー分けであったり、DDDのパターンであったりするんですが、基本的には「SOLID原則」や「カプセル化」のようなオブジェクト指向プログラミングにおける原則を守ることが重要だと思っています。

ルールを守ったうえで細部をどう実装するかについては好みの部分が出てきますが、新しい設計思想を採用する場合には、その思想の根幹にあるルールを理解しておくことが重要です。


## 参考

https://www.shoeisha.co.jp/book/detail/9784798126708  
https://www.ritolab.com/posts/185  
https://zenn.dev/kohii/articles/e4f325ed011db8  
https://zenn.dev/mpyw/articles/ce7d09eb6d8117  
https://qiita.com/fuwasegu/items/f778a1067c941dc8baea  
https://blog.ytake.jp.net/entry/2015/12/16/011812  