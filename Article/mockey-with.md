---
title: Mockeryを使った引数の検証方法まとめ
tags: [test]
createDate: 2023-11-13
updateDate: 2023-11-13
slug: mockery-with
---

## はじめに

プログラミングをしていて、誰しもわからない箇所やメソッドの使い方がわからなくて調べる事があると思います。  
今回、私は一度引っかかったことにもう一度引っかかって、さらに思い出すまでに時間がかかったのでメモとして残しておきます。  

また、mockeryはとても直感的に使用できるライブラリではあるんですが、日本語での情報が少なく、今回問題を解決しようとして自分が以前書いた記事を見直すこともあったので、改めて知見を日本語で残しておこうと思います。

勝手に高専出身勢は全員英語出来ないと思っているので日本語の記事があると助かりますね。

ChatGPTに誤字脱字の校閲してもらったら指導を受けました。
>読者層の想定: 記事の冒頭で「勝手に高専出身勢は全員英語出来ないと思っている」という記述があります。これは一般化の可能性があり、読者によっては不快に感じる可能性があります。対象読者層をより広くするためには、このような前提を排除する表現が適切かもしれません。

以前書いた記事はこちらです。

[Mockeryの基本的な使い方](https://www.lyricrime.com/posts/mockery/)

また、mockeryの公式ドキュメントの翻訳は以下になります。

https://readouble.com/mockery/1.0/ja/index.html

## 結論

- 基本的にモックの引数を検証する場合において、引数がオブジェクトの場合はMockery::onを使用しましょう。
- 引数が複数の場合は、引数ごとにMockery::onを渡す必要があります。

## 今回の問題

今回の問題はmockeryを使用した引数の検証についてです。  

例として、今回は以下のようなケースを考えます。

```php
class User
{
    public function __construct(
        private readonly string $userIdentifier,
        private string $userName,
    ) {
    }

    public function userIdentifier(): string
    {
        return $this->userIdentifier;
    }

    public function userName(): string
    {
        return $this->userName;
    }

    public function changeUserName(string $userName): void
    {
        $this->userName = $userName;
    }
}

class UserRepository
{
    public function findById(string $identifier): User
    {
        // 再構築処理
    }

    public function save(User $user): void
    {
        // 　永続化処理
    }
}
```

これらのクラスを利用するユースケースを考えます。

```php
class ChangeUserName
{
    public function __construct(
        private UserRepository $userRepository,
    ) {
    }

    public function execute(string $userIdentifier, string $userName): void
    {
        $user = $this->userRepository->findById($userIdentifier);
        $user->changeUserName($userName);
        $this->userRepository->save($user);
    }
}
```

このユースケースに対してテストを書いていきます。

```php
class ChangeUserNameTest
{
    public testExecute(): void
    {
        $userRepository = \Mockery::mock(UserRepository::class);

        // findByIdの動作を定義します。
        $userRepository->shouldReceive('findById')
            ->with('user-identifier')                               // 引数の検証
            ->andReturn(new User('user-identifier', 'user-name'));  // 戻り値の指定

        $userRepository->shouldReceive('save')                      // 素直に書けばこれで引数の検証ができる?
            ->with(new User('user-identifier', 'new-user-name'));

        $changeUserName = new ChangeUserName($userRepository);
        $changeUserName->execute('user-identifier', 'new-user-name');
    }
}
```

ところが、このテストは失敗します。

公式ドキュメントを確認します。  

[引数のバリデーション](https://readouble.com/mockery/1.0/ja/argument_validation.html)

> オブジェクトの引数のマッチングでは、Mockeryは厳密な===比較だけを行いますので、全く同じ$objectのみ一致します。

PHPにおいて、厳密な比較の場合、オブジェクトの場合は同じインスタンスである必要があります。今回のテストでは、`findById`で取得したオブジェクトと`save`に渡すオブジェクトは別のインスタンスになっているため、テストが失敗しています。

## 解決策

今回の問題を解決するには、Mockery::onを使用します。

公式ドキュメントにも記載があるのですが、いくつかバリエーションを紹介しておきます。

### 単純なMockery::on

まずは冒頭のケースでのMockey::onの使用例を紹介します。

```php
public function testExecute(): void
{
    // ~~省略~~
    $userRepository->shouldReceive('save')
        ->with(\Mockery::on(function (User $user) {             // 無名関数の引数には実際にメソッドに渡される引数を指定します。
            $this->assertSame('user-identifier', $user->userIdentifier());
            $this->assertSame('new-user-name', $user->userName());
            return true;
            // 検証に成功したかどうかをboolで返します。今回の場合は検証に失敗するとassertSame関数が例外を投げるので、常にtrueを返します。
        }));
    // ~~省略~~
}
```

### assertSameを使用せずに引数の確認

Mockery::onを使用する場合は、無名関数内でboolを返せばいいので、比較を自分で書いても大丈夫です。

公式にもこちらの方法の記載があります。

また、$thisを使用しない用に書くとstaticを付与する事ができるようになります。

```php
public function testExecute(): void
{
    // ~~省略~~
    $userRepository->shouldReceive('save')
        ->with(\Mockery::on(static function (User $user) {          // 無名関数の引数には実際にメソッドに渡される引数を指定します。
            return $user->userIdentifier() === 'user-identifier';   // 例えば、IDのみを比較したい場合。
        }));
    // ~~省略~~
}
```

### メソッドが複数引数を取る場合

今回私が引っかかったのは、メソッドが複数の引数を取る場合でした。

次のようなユースケースのテストを考えます。

```php
class CreateUser
{
    public function __construct(
        private UserFactory $userFactory,
        private UserRepository $userRepository,
    ) {
    }

    public function execute(UserName $userName, Email $email): void
    {
        $user = $this->userFactory->create($userName, $email);
        $this->userRepository->save($user);
    }
}

class UserFactory
{
    public function create(UserName $userName, Email $email): User
    {
        // IDの採番
        // エンティティの作成
    }
}
```

このユースケースに対してテストを書いていきます。

先程と異なる点は、ID、Emailといったパラメータがオブジェクトとなっています。

```php
class CreateUserTest
{
    public function testExecute(): void
    {
        $userFactory = \Mockery::mock(UserFactory::class);
        $userRepository = \Mockery::mock(UserRepository::class);

        $userFactory->shouldReceive('create')
            ->with(
                \Mockery::on(static function (UserName $userName) {
                    return $userName->value() === 'user-name';  // 第一引数の検証
                }),
                \Mockery::on(static function (Email $email) {
                    return $email->value() === 'user-email';    // 第二引数の検証
                })
            ) // withの引数には、引数の数だけMockery::onを渡す必要があります。
        ->andReturn(new User('user-identifier', 'user-name'));  // 戻り値の指定

        $userRepository->shouldReceive('save')
            ->with(\Mockery::on(function (User $user) {
                $this->assertSame('user-identifier', $user->userIdentifier()); // Factoryから返却されたUserIdであることを検証します
                $this->assertSame('user-name', $user->userName());
                $this->assertSame('user-email', $user->email());
                return true;
            }));

        $createUser = new CreateUser($userFactory, $userRepository);
        $createUser->execute(new UserName('user-name'), new Email('user-email'));
    }
}
```

このように、mockery::onを使用することでかなり柔軟な引数の検証が可能になります。

## まとめ

- 基本的にモックの引数を検証する場合において、引数がオブジェクトの場合はMockery::onを使用しましょう。
- 引数が複数の場合は、引数ごとにMockery::onを渡す必要があります。

## 所感

オブジェクトに対する引数の検証は、設計をきっちりしようとするとかなり頻出するパターンです。  
今回の例のように、Repositoryを採用した場合は、抽象化されるため、検証内容が以下のように変わります。  

実際のDBに意図した値が保存されていること => saveメソッドに渡されるオブジェクトが意図したものであること

Mockeryを使用する上で、withを使用した様々なテスト実装パターンが存在しますが、Mockery::onがあれば、オブジェクトの検証はほぼカバーできると思います。
