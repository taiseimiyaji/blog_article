---
title: 「ドメイン駆動設計入門」を読む その2 ユースケースを組み立てるためのパターン編
tags: [BookReview]
createDate: 2022-11-30
updateDate: 2022-11-30
slug: ddd-init-02
---


## はじめに

[前回の記事](https://taisei-miyaji.hatenadiary.com/entry/2022/11/26/181405)にて、ドメイン駆動設計への入門として、ドメインオブジェクトを紹介しました。
今回は実際にドメインオブジェクトを利用してユースケースを組み立てていきます。  
そのためのステップとして、先にいくつか便利なパターンを紹介しておきます。  

今回紹介するパターンは

- Repository
- Factory
- Service

の3つです。

ちなみに、混乱を避けるために説明のスコープを絞り、クリーンアーキテクチャとは分けて書いたつもりですが、一部クリーンアーキテクチャの思想がサンプルに含まれている点がありますのでご注意ください。

## 対象書籍
[ドメイン駆動設計入門 ボトムアップでわかる! ドメイン駆動設計の基本](https://www.amazon.co.jp/%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E9%A7%86%E5%8B%95%E8%A8%AD%E8%A8%88%E5%85%A5%E9%96%80-%E3%83%9C%E3%83%88%E3%83%A0%E3%82%A2%E3%83%83%E3%83%97%E3%81%A7%E3%82%8F%E3%81%8B%E3%82%8B-%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E9%A7%86%E5%8B%95%E8%A8%AD%E8%A8%88%E3%81%AE%E5%9F%BA%E6%9C%AC-%E6%88%90%E7%80%AC-%E5%85%81%E5%AE%A3/dp/479815072X)

今回も前回同様、上記の書籍を参考に進めていきますが、今回の記事はより個人的な見解(ほぼ個人的見解)が含まれます。
書籍にて解説されている内容を知りたい方はぜひ購入してみてください。(kindleだとセールで結構安くなっていたりします。)

## Repository
Repositoryという単語の意味は、「保管庫」です。

ソフトウェアにおいてはドメインオブジェクトを作成したとしても、メモリ上に展開されるので、プログラムが終了した場合は消えてしまいます。
消えてしまっても困るデータについてはデータストアに永続化する必要があります。  
リポジトリはデータを永続化するという責務を凝集するためのオブジェクトといえます。

Repositoryパターンを採用した場合、オブジェクトのインスタンスを永続化したい場合はデータストアに直接書き込むのではなく、リポジトリにインスタンスの永続化を依頼します。
また、データストアに永続化したデータからインスタンスを再構築したい場合にもリポジトリにデータの再構築を依頼します。

リポジトリはドメインオブジェクトではありませんが、ドメインを表現したコードを実現するために、データストアへの具体的なアクセス処理をすべて隠蔽する事ができます。ドメインを表現するための構成要素として欠かせないものです。

リポジトリは具体的にどういった内容を隠蔽するのかについて考えてみます。  
リポジトリの責務はドメインオブジェクトの**永続化**と**再構築**を行うことです。  
また、保存先であるデータストアに基づく具体的な処理を隠蔽します。  
データストアといえばMySQLなどのRDBを想像するかもしれませんが、他のDBMSに変わったり、単純にファイルに保存したり、スプレッドシートのようなサービスに保存したりと保存先はいろいろ考えられます。保存先が変更となった場合でもドメインコードに変更がないように、つまり永続化に関連する処理にドメインコードが依存しないように隠蔽します。

では具体的にリポジトリがどういう処理を担うのか、インターフェースから考えてみます。
今回は例として、[前回の記事](https://taisei-miyaji.hatenadiary.com/entry/2022/11/26/181405)同様システムを利用するユーザーというドメインについて考えてみます。

```PHP
interface RepositoryInterface
{
    /**
     * 再構築を担う処理
     */
    public function findById(UserIdentifier $id): User;

    /**
     * 永続化を担う処理
     */
    public function save(User $user): void;
}

```
ここで永続化したいユーザーのエンティティは以下のようなものを想定しています。

```PHP
class User
{
    // UserIdentifierというValueObjectを保持するためのプロパティ
    private UserIdentifier $id;

    // UserNameというValueObjectを保持するためのプロパティ
    private UserName $name;

    public function __construct(
        UserIdentifier $id,
        UserName $name
    ){
        $this->id = $id;
        $this->name = $name;
    }

    public function id(): UserIdentifier
    {
        return $this->id;
    }

    public function name(): UserName
    {
        return $this->name;
    }
}
```

続いて、Repositoryを利用する側を考えてみます。  
今回はユーザー情報の更新について考えてみます。  
例えば、下記のコードでユーザー名を変更します。

```PHP
class UpdateUser
{
    private RepositoryInterface $repository;

    // Laravelのサービスコンテナを用いたDIを行う。
    // 今回の説明において重要な箇所ではないのでDIに馴染みが無い方は一旦読み飛ばしても構いません。
    public function __construct(RepositoryInterface $repository)
    {
        $this->repository = $repository;
    }

    public function process(UserIdentifier $id, UserName $name): User
    {
        // データストアからインスタンスの再構築を行う。
        $user = $this->repository->findById();

        // エンティティのもつ名前変更の振る舞いを呼び出して名前を変更する。
        $user->changeName($name);

        // データストアへの永続化を行う。
        $this->repository->save($user);
    }
}
```
UserIdentifier,UserNameは必要なバリデーションを実装した、値の性質を持ったValueObjectです。  
Userエンティティは名前を変更するための振る舞い「changeName」を持っています。  
Repositoryには再構築のための振る舞い「findById」と永続化のための振る舞い「save」を定義します。  

こうすることで、サンプルコードの様に具体的なデータストアの操作をすべてRepositoryに隠蔽し、ドメインを表現したコードにすることができます。

ドメインのコードにDBアクセスに関連するコードが氾濫しないことを確認できたので具体的なDBに対する操作はどう書くのか考えてみます。  

今回はLaravelのORMであるEloquentを例にしますが、ORMがなんであれ、接続先DBがなんであろうと永続化と再構築という処理がドメインコードから隠蔽されていれば問題ないです。

以下サンプルではLaravelのEloquentモデルとして`User`を使用し、同じ`User`という名前でエンティティを作成しているため違いに注意してください。

```PHP
class Repository extends RepositoryInterface
{
    /**
     * Eloquentモデル
     */
    private \App\Models\User $user;


    public function __construct(\App\Models\User $user)
    {
        $this->user = $user;
    }

    /**
     * IDに紐づくデータをUserエンティティに詰めて返す。
     */
    public function findById(UserIdentifier $id): User
    {
        $query = $this->user->newQuery()
            ->where('id', '=', (string)$id)
            ->first();
        return new User(
            new UserIdentifier($query->getAttribute('id')),
            new UserName($query->getAttribute('name'))
        );
    }

    /**
     * エンティティを永続化する。
     */
    public function save(User $user): void
    {
        $query = $this->user
            ->fill([
                'id' => (string)$user->id(),
                'name' => (string)$user->name()
            ])
            ->save();
    }
}
```

このような形で、利用するフレームワークの機能を使用したりしてDBへのアクセスロジックをRepository内に記載します。  
重要なのはフレームワークの機能やDBアクセスの複雑なロジックをドメインを表現している層に漏洩させないことです。  
開発初期にDBがまだ選定されていない場合や運用中に変更となった場合でもその変更はドメインロジックには影響しなくなります。  
また、テストを行う際にはリポジトリをモック化することでドメインロジックのテストがより容易になります。

## Factory

次に紹介するパターンは「Factory」パターンです。直訳すると工場という意味があります。

[前回の記事](https://taisei-miyaji.hatenadiary.com/entry/2022/11/26/181405)で作成したようなドメインオブジェクトはドメインモデルを反映させるが故にときに複雑なものとなります。
複雑なドメインオブジェクトの生成に関する知識をまとめたのが「Factory」です。

これまでと同じくユーザーというドメインモデルについて考えます。ユーザーエンティティの生成処理を担う`UserFactory`のもつ振る舞いを考えてみます。  
ユーザーエンティティにはIDを持ちますがこのIDの採番処理はドメインを操作する上では直接関係なく、生成時に行う処理なのでFactoryでもつようにしてみましょう。

インターフェースは以下のようユーザーエンティティの作成処理を定義しておきます。

```PHP

interface UserFactoryInterface
{
    public function createUser(UserName $name): User
}

```

今回はIDの採番に[ULID](https://github.com/ulid/spec)形式を採用します。
IDの生成にはLaravelに含まれるSymfonyというライブラリを使用します。

```PHP
use Symfony\Component\Uid\Ulid;

class UserFactory implements UserFactoryInterface
{
    public function createUser(UserName $name): User
    {
        $id = Ulid::generate();
        $user = new User(
            $id,
            $name
        );
    }
}
```

今回はフレームワークの機能に頼ったのでシンプルな処理になりましたが、他にもエンティティの生成時に発生する処理をFactoryに隠蔽できます。  
例えばエンティティの生成時刻をエンティティ自身がもつような場合、現在時刻を取得してエンティティに設定するのはFactoryに書くべきです。  

RepositoryのときはDBに依存しないように処理を記載しました。Factoryを採用することでIDの採番や作成時刻等をDBに依存せずに記述できます。
ドメイン駆動設計においては具体的なDBMSに依存することを避けることがドメインロジックに集中して取り組むために必要です。

ここで考えられる疑問点として、インスタンスの生成はコンストラクタで行うのでFactoryを使用せずにコンストラクタに書けばよいのでは?というものです。
この疑問に対する答えは、処理が複雑なものはFactoryに記載するべき、です。  
コンストラクタ内で他のオブジェクトを生成しているような場合はまずFactoryの利用を検討します。  
特に、外部のフレームワークやライブラリに依存する場合は依存関係が発生するのでFactoryを使用しない場合にドメインルールがフレームワーク依存になります。つまりフレームワークを変更する場合にドメインオブジェクトのコードに変更が発生してしまいます。  
このような自体を避けるためにFactoryを利用しましょう。

大切なのは**思考停止するのではなく根拠を持ってFactoryを利用するかどうか判断する**ことです。

## Service

ドメイン駆動設計という文脈においてServiceと呼ばれるものには以下の二種類があります。

- ドメインサービス
- アプリケーションサービス

名前は似ていますが、この２つは明確な違いがあります。違いを意識しながら読み進めてもらえればと思います。


### ドメインサービス

ValueObjectやエンティティには振る舞いが記述されます。  
例えばユーザー名の最大文字数の制限等がそれにあたります。  
しかし、ValueObjectやエンティティに記述すると不自然になってしまう振る舞いが存在します。

例えば、ユーザーの重複チェックです。  
ユーザーエンティティが重複チェックを行うという振る舞いを持つ場合、重複の有無を自身に対して問い合わせることは不自然です。値の重複チェックは値を利用した別のオブジェクトが持つほうが自然になります。

この際に利用するのがドメインサービスになります。

```PHP
interface UserServiceInterface
{
    /**
     * 重複をチェックする処理
     */
    public function isExists(User $user): bool
}
```

具体的な処理については割愛しますが、重複の確認を`UserService`というドメインサービス内で行います。
このサービスを用いてユーザーの作成処理を書いてみます。また、先述したFactoryやRepositoryも使用してみましょう。

```PHP
class CreateUser
{
    private UserRepositoryInterface $repository;

    private UserFactoryInterface $factory;

    private UserServiceInterface $service;

    // Laravelのサービスコンテナを用いたDIを行う。
    // 今回の説明において重要な箇所ではないのでDIに馴染みが無い方は一旦読み飛ばしても構いません。
    public function __construct(
        UserRepositoryInterface $repository,
        UserFactoryInterface $factory,
        UserServiceInterface $service
        )
    {
        $this->repository = $repository;
        $this->factory = $factory;
        $this->service = $service;
    }

    public function process(UserName $name): User
    {
        // IDはFactoryにて生成されるためユーザー名を渡すとエンティティが返される。
        $user = $this->factory->createUser($name);

        // ドメインサービスを用いて例外チェックを行い、重複している場合は例外をスローする。
        if(!$this->service->isExists())
        {
            throw new NotFoundException();
        }

        // データストアへの永続化を行う。
        $repository->save($user);

    }
}
```

ドメインモデルを実装する際にはドメインオブジェクトに実装すると不自然になる振る舞いが必ず存在します。  
これは特に複数のドメインオブジェクトを横断するような操作に多く見られます。  
そんなときにはドメインサービスの利用を検討してください。

### アプリケーションサービス

アプリケーションサービスを一言でいうと、**ユースケースを実現するオブジェクト**です。  
実際に私の所属しているチームではこのアプリケーションサービスをユースケースと呼ぶ事が多いです。

ここで「ドメイン」と「アプリケーション」という命名について考えてみます。  
「アプリケーション」とは一般的には利用者の目的を達成するためのプログラムのことを指します。  
ValueObjectやエンティティといったドメインオブジェクトは「ドメイン」を表現するためのものです。
ドメインを表現してもそれだけでは利用者の目的は達成されません。  
ドメインオブジェクトを目的に沿って操作する必要があり、それはまさしく利用者の目的を達成するための「アプリケーション」といえます。

先程から例に上げているユーザー機能においては、「ユーザーを作成する」、「ユーザー情報を更新する」、「ユーザーを削除する」等がユースケースにあたります。   
先程紹介した以下のコードはまさしくアプリケーションサービスそのものです。


```PHP
class CreateUser
{
    private UserRepositoryInterface $repository;

    private UserFactoryInterface $factory;

    private UserServiceInterface $service;

    // Laravelのサービスコンテナを用いたDIを行う。
    // 今回の説明において重要な箇所ではないのでDIに馴染みが無い方は一旦読み飛ばしても構いません。
    public function __construct(
        UserRepositoryInterface $repository,
        UserFactoryInterface $factory,
        UserServiceInterface $service
        )
    {
        $this->repository = $repository;
        $this->factory = $factory;
        $this->service = $service;
    }

    public function process(UserName $name): User
    {
        // IDはFactoryにて生成されるためユーザー名を渡すとエンティティが返される。
        $user = $this->factory->createUser($name);

        // ドメインサービスを用いて例外チェックを行い、重複している場合は例外をスローする。
        if(!$this->service->isExists())
        {
            throw new NotFoundException();
        }

        // データストアへの永続化を行う。
        $repository->save($user);

    }
}
```

ドメインサービスとアプリケーションサービスは対象となる領域が「ドメイン」なのか「利用者の目的を達成すること」なのかという点が異なる以外は本質的には同じものです。ただし、その領域をきちんと分け、ドメインのルールがアプリケーションサービスに流出しないように実装することでドメインの変更をドメインオブジェクトのみに反映すれば良くなります。変更容易性の確保のためにも意識して振る舞いを実装しましょう。

## まとめ
**Repository** ... データアクセスに関するロジックをまとめるためのパターン  
**Factory** ... ドメインオブジェクトの生成に関するロジックをまとめるためのパターン  
**Service** ... ドメインオブジェクトに実装するのが不自然なものをドメインサービス、アプリケーションの目的を達成するためのロジックをアプリケーションサービスに実装するためのパターン  

## 所感
[前回の記事](https://taisei-miyaji.hatenadiary.com/entry/2022/11/26/181405)と合わせて、ドメイン駆動設計におけるドメインモデルをコードに反映するためのパターンをいくつか紹介しました。
これらは**軽量DDD**と呼ばれます。  
ドメイン駆動設計において大事な要素は、ドメインモデルの継続的な改善です。そこに着手するための準備として、今回の記事がお役に立てばと思います。
より詳しいパターンの説明や抱くであろう疑問の多くは書籍で解説されている部分も多くありますのでよければ参考にしてください。

書籍のない内容に沿っていますが、自分なりの解釈がほとんどです。まだまだ理解しきれていない部分もありますが、こうしてアウトプットすることと日々の業務に向き合っていくことで引き続き設計について身につけて行ければと思います。

実際の業務ではクリーンアーキテクチャという考え方とドメイン駆動設計の要素をハイブリットして取り入れていますが、改めてドメイン駆動設計における考え方を整理しておくことと、目的を再認識できるいい記事になったなと感じています。

次回は今回の記事では紹介しきれなかった部分をTips的に紹介しようと思っています。