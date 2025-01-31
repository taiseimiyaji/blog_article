---
title: Mockeryで学ぶMockとSpyの違いとつかいかた
tags: [UnitTest]
createDate: 2025-01-31
updateDate: 2025-01-31
slug: unittest-mock-spy
---

## はじめに

単体テストにおいてテストダブルを用いることは、コードの設計品質を保つうえで、抽象に依存させる大きな動機になるため、コード本体の品質を上げることに繋がります。

テストダブルのなかでも、外部リソースや他のクラスへの依存を切り離せる「モック（Mock）」と、呼び出しを記録して後から検証できる「スパイ（Spy）」はしばしば混同されがちな概念です。

PHPでモックを扱うライブラリとして有名なMockeryでも、モックとスパイはユニットテストを設計する際の要となります。

本記事では、Mockeryを例に挙げつつ、モックとスパイの違いや使い分け、具体的なコード例を交えて分かりやすく解説してみます。

ライブラリとしての使い方の多くは過去の記事が参考になると思います。

- [Mockeryを使った引数の検証方法まとめ](https://www.lyricrime.com/posts/mockery-with/)
- [Mockeryの基本的な使い方](https://www.lyricrime.com/posts/mockery/)

## 注意

記事執筆時点のMockery最新バージョンである1.6.12では、Spyオブジェクトに関するタイプヒントがうまく動作しないバグが報告されています。

Spyで生成したオブジェクトに対して呼び出し回数の検証を行う`times`等のメソッドがIDE等では認識されないケースがありますが、問題なく動作します。

詳細は下記のGitHub Issueを参照してください。

- [1.6.12 スパイメソッドのタイプヒントの問題 · 問題 #1421 · mockery/mockery](https://github.com/mockery/mockery/issues/1421)

## Mockeryとは

MockeryはPHPのモックオブジェクト生成ライブラリで、PHPUnitなどのテストフレームワークと組み合わせて使用されることが多いです。

Laravel等のフレームワークでも標準で採用されています。

[Mocking - Laravel 11.x - The PHP Framework For Web Artisans](https://laravel.com/docs/11.x/mocking)

テスト対象の依存関係をモック化し、メソッド呼び出し回数や引数、戻り値などを詳細に設定・検証できます。

可読性の高いメソッドチェーンによる記述方式を採用しており、可読性が高く、直感的な操作が可能なため、PHPUnit標準のモックと比較しても学習コストが比較的低いのが特徴です。

Mockeryを使うと、外部APIやデータベースなどへの依存を極力排除しながらテストを実施できるようになります。

## Mockとは?

モックは、テスト対象が依存する外部要素やオブジェクトを「模倣」し、あらかじめ設定した条件に沿ってメソッド呼び出しの可否や戻り値を制御するテストダブルの一種です。

最大の特徴は「期待する呼び出し回数や引数、戻り値などを事前に定義し、その通りに呼び出されるかを検証する」点にあります。

例えば、外部APIを呼び出すメソッドをテストしたい場合、実際にネットワーク通信を行う代わりにモックを使って、「どの引数で何回呼び出され、その結果として何を返すか」を決めておけば、外部リソースに依存しない安定したテストが可能になります。

Laravelではライブラリを使わずともサービスコンテナ等の仕組みでモックの実装に切り替えることもできますが、テストケースの中でモックを用意したいケースが多く、そういった場合にはMockeryを使うことで簡単にモックオブジェクトを作成できます。

モックを使うと、呼び出しの期待値が満たされない場合にテストが失敗するようになるため、依存するメソッドが「本当に期待された通りに使われているか」を厳密にチェックできます。

言い換えると、想定した引数が渡されているか、想定した回数だけ呼び出されているかといったことをテストすることができます。

一方で、事前に呼び出しの振る舞いを細かく設定するため、実装の変更に合わせてテストの修正頻度が増えることがあります。テスト対象の責務とモック化の範囲を見極めたり、テストコードを共通化するなどしてコストを下げる必要があります。

## Spyとは?

スパイもモックと同様にテストダブルの一種ですが、「呼び出しを記録し、後から参照して検証できる」ことを重視しています。

モックが事前に設定した期待値に沿った振る舞いを強制するのに対し、スパイは実際にメソッドを（部分的にまたは完全に）実行し、最終的に「何回、どのような引数で呼び出されたか」をテスト終了時などにチェックします。

呼び出し回数や引数を事前に縛りすぎたくない場合や、実際の処理をできるだけ動かしつつ呼び出し履歴のみを確認したい場合に効果的です。

例えばログ出力処理をテストする際に、ログを「実際に吐き出すかどうか」を確認するだけで十分ならスパイが向いています。

ログのメソッドが何回呼ばれたかをテスト後に調べられればよいので、厳密に呼び出し順序や引数を規定する必要がなければ「あとで呼び出し記録をチェックする」アプローチで事足ります。

スパイはモックと比べて「実際の処理を動かす」ため、テスト対象の振る舞いが変わらないことを確認しやすいという利点があります。

似たようで違うオブジェクトですが、Laravelで利用する場合はどちらもサービスコンテナを利用して抽象に対してバインドして利用することができます。

スパイやモックは対象の抽象クラスやインターフェースの実装としてオブジェクトが生成されますが、クラスに対してもスパイやモックを生成することができます。ただし、その仕組み上finalクラスやprivateメソッドに対してはスパイやモックを生成することができません。  

## Mockeryでのモックの利用例

```php
use Mockery;
use PHPUnit\Framework\TestCase;

class UserServiceTest extends TestCase
{
    public function tearDown(): void
    {
        Mockery::close();
    }

    public function testGetUserById()
    {
        $mockRepository = Mockery::mock('UserRepository');
        $mockRepository
            ->shouldReceive('findUserById')
            ->once()
            ->with(123)
            ->andReturn([
                'id' => 123,
                'name' => 'Taro'
            ]);

        $userService = new UserService($mockRepository);
        $result = $userService->getUserById(123);

        $this->assertSame(123, $result['id']);
        $this->assertSame('Taro', $result['name']);
    }
}
```

上記のように、shouldReceive('findUserById') で呼び出しメソッドを指定し、once() で呼び出し回数、with(123) で引数、andReturn(...) で戻り値を定義します。期待した通りの呼び出しが行われない場合はテストが失敗し、モックとしての役割を果たします。

この例ではコンストラクタインジェクションを利用しています。直接引数に指定していますが、Laravelのサービスコンテナと合わせて利用したい場合は`$this->app->instance()`を利用してインスタンスを直接バインドすることができます。  

## MockeryのPartial Mock

Mockeryでは、スパイに近いふるまいを実現するためにPartial Mockという機能が用意されています。

makePartial()を使ってモック化したいメソッドだけを指定し、ほかは元の実装を呼び出すようにできます。

さらに、実際に呼び出しがあったかどうかを後で検証する shouldHaveReceived() などのAPIも備えています。

Spyとは別物ですが、オブジェクトの1部分だけをモック化するという場合はこちらを利用すると便利です。

```php
use Mockery;
use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    public function tearDown(): void
    {
        Mockery::close();
    }

    public function testPartialMockAsSpy()
    {
        $partial = Mockery::mock('MyClass')->makePartial();
        $partial->shouldReceive('calculate')->andReturn(50);

        $result = $partial->process(10, 5);

        $partial->shouldHaveReceived('calculate')->once();
        $this->assertSame(50, $result);
    }
}
```

この例では、calculate メソッドのみモック化し、他のメソッドは通常の実装どおりに動作させています。

そしてテスト終了時に `shouldHaveReceived('calculate')` を呼び出すことで、想定どおりにメソッドが呼び出されたかを確認しています。

呼び出し記録をあとでチェックする点がモックとは違います。

呼び出す場所が違うということは、実際にテスト中に検証に失敗した場合にエラーが発生するタイミングが異なるということです。

モックの場合は意図していない呼び出しの場合は即座にエラーが発生しますが、スパイの場合はテスト終了後、検証が呼び出されたタイミングでエラーが発生します。

## MockeryでのSpyの利用例

Mockeryには、モック(Mock)のほかに「スパイ(Spy)」として機能するオブジェクトを簡単に生成するメソッドとして Mockery::spy() が用意されています。

スパイは、呼び出しの記録を後から検証するアプローチを得意とするテストダブルです。

モックと同じように依存関係を差し替えられる一方で、テストの中で「実際に呼び出されたか」をあとから確認したい場合に活用されます。

```php
use Mockery;
use PHPUnit\Framework\TestCase;

class MyClass {
    public function doSomething($param)
    {
        // 実装（テスト内では実際に動かさない場合も）
    }
}

class SpyExampleTest extends TestCase
{
    public function tearDown(): void
    {
        // テスト完了後にMockeryをクローズ
        Mockery::close();
    }

    public function testSpyUsage()
    {
        // 1) Spyを生成する
        $myClassSpy = Mockery::spy('MyClass');

        // 2) テスト対象（例として直接呼び出し）でメソッドを呼ぶ
        $myClassSpy->doSomething('Hello');

        // 3) テスト終了後に呼び出し状況を検証する
        $myClassSpy->shouldHaveReceived('doSomething')
            ->once()
            ->with('Hello');
    }
}
```

1. スパイの生成
Mockery::spy('MyClass') で、MyClass のスパイを生成しています。このオブジェクトを使って実際のメソッドを呼び出すと、その呼び出し情報（回数や引数など）が記録されます。

2. メソッドの呼び出し
テストの中で doSomething() を呼び出すと、テスト完了時点で「どのようなメソッドが何回呼ばれたか」がスパイ内部に保持されます。

3. 呼び出しの検証
shouldHaveReceived('doSomething') を利用して、「doSomething メソッドが呼ばれたかどうか」「何度呼ばれたか」「どんな引数が渡されたか」を後から検証します。期待通りでなければテストが失敗し、呼び出し回数や引数が確認できます。

また、イベントの発火の監視も可能です。

```php
$dispatcherSpy = Mockery::spy('EventDispatcher');

// テスト対象のメソッド内で $dispatcherSpy->dispatch('UserCreated') が呼ばれる想定
$userService->createUser($dispatcherSpy);

$dispatcherSpy->shouldHaveReceived('dispatch')
    ->once()
    ->with('UserCreated');
```

この例だと、ユーザーを作成したあとで`dispatch`メソッドが`UserCreated`イベントを1度だけ発火することを検証しています。

モックと違い、呼び出し後の検証になるため、「必ず1回だけ」呼び出される、というよりは「本当に呼ばれたか」を検証したい場合に向いています。

同じ考え方で、ログを吐き出すメソッドをスパイで検証すれば、ログを実際に吐き出す必要はないが、吐き出す内容を検証する事ができます。

なお、私も勘違いしていたんですが、Spyにクラス名を指定して生成した場合、デフォルトで **実際のメソッドは呼ばれない** ようになっています。

呼び出し履歴や回数こそ記録されますが、実際の処理は動作しません。

Spyの生成時に既存のインスタンスを引数に渡した場合は、実際のメソッドをspy化するため、実際の処理が動作します。

## パーシャルモックとスパイの使い分け

Mockeryでは、クラスの一部のメソッドだけモック化し、他のメソッドは実際の実装を呼び出す「パーシャルモック」も提供しています。一見するとスパイに似ていますが、以下のような使い分けが考えられます。

- スパイ:
  - Mockery::spy('ClassName') などで生成し、主に「呼び出し履歴を検証」することが目的。事前に戻り値や呼び出し回数を縛らず、後で shouldHaveReceived() を使ってチェックする。
- パーシャルモック:
  - Mockery::mock('ClassName')->makePartial() のように生成し、「特定のメソッドだけモック化」したうえで、ほかのメソッドは実際に呼ばせることができる。
    - 例えば、計算処理やDBアクセスをするメソッドはモック化しつつ、そのほかのユーティリティ的なメソッドは実際に動かして結果を検証する、といった使い方ができる。

## まとめ

Mockery では、モック的な「事前定義」とスパイ的な「事後検証」を同時に活用するケースが多いです。

shouldReceive() と shouldHaveReceived() を組み合わせることで、厳密な呼び出しチェックと呼び出し履歴の後追い確認の両面をカバーできます。

つまり、「事前の厳密性を優先するか、事後の柔軟な検証を優先するか」という観点で使い分けを考えるといいかなと思います。

