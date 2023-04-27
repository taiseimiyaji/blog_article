---
title: エラーハンドリングをクリーンアーキテクチャで書く場合に意識すること
tags: [Laravel]
createDate: 2023-04-27
updateDate: 2023-04-27
slug: error-handle-clean-arch
draft: false
---

## はじめに

業務においてクリーンアーキテクチャを意識してコードを日々書いていますが、「これってクリーンアーキテクチャの考えに沿うためにはどう書けば良いんだろう」と悩むことがあります。  
今回はエラーハンドリングについて、クリーンアーキテクチャに則って考えてみたいと思います。  
あくまで筆者一個人のいわゆる「ぼくのかんがえたさいきょうのエラーハンドリング」なのでエラーハンドリング時の一意見として参考になれば幸いです。

## 想定読者

- **Webのバックエンド開発**においてクリーンアーキテクチャを意識してコードを書いているがエラーハンドリングに悩んでいる人
- エラーハンドリングをするメリットを知りたい人

## まとめ

- 基本的には型で例外をハンドリングするべきなので自作型を作ろう
- ドメイン層にHttpExceptionを書くな
- ドメイン層はプレゼンテーション層を意識しないため、エラーメッセージをAction層でコントロールするといいかも

## 前提 クリーンアーキテクチャについて

![クリーンアーキテクチャ](/images/clean_arch.png)

詳細は以前の記事でも紹介したのでそちらを御覧ください。

今の私の理解だと、クリーンアーキテクチャとは、「ソフトウェアにおいてもっとも重要なドメインルールを中心に考え、ドメインルールに依存するように依存の方向性を制限した上で適切に責務を分割し、テスタブルで交換可能なソフトウェア構成を実現するための設計手法」という捉え方をしています。

今回はこの前提に沿ってエラーハンドリングについて考えていきます。

今回は例題として、ユーザーのCRUDについて考えていきます。


## Entity(Entities)

まずはコアとなるドメインオブジェクト、Entityについてです。

今回はユーザーを例にするのでコアとなるエンティティはユーザーを表現します。

```php
class User
{
    private UserId $userId;
    private UserName $userName;
    private UserEmail $userEmail;

    public function __construct(
        $userId,
        $userName,
        $userEmail
    )
    {
        $this->userId = $userId;
        $this->userName = $userName;
        $this->userEmail = $userEmail;
    }

    public function userId(): UserId
    {
        return $this->userId;
    }

    public function userName(): UserName
    {
        return $this->userName;
    }

    public function userEmail(): UserEmail
    {
        return $this->userEmail;
    }

    public function toArray(): array
    {
        return [
            'userId' => (string)$this->userId,
            'userName' => (string)$this->userName,
            'userEmail' => (string)$this->userEmail
        ];
    }
}

```

現状はエンティティの振る舞いとして特段思いつくエラーがないので次に進みます。

## ValueObject(Entities)

次に表現するのはValueObjectです。

エンティティの例で出てきた`UserId`や`UserName`などのクラスを指します。

たとえば、`UserName`の場合は以下のようなクラス定義になります。

```php
class UserName
{
    public const MAX_LENGTH = 20;

    private string $value;

    public function __construct(string $value)
    {
        $this->validate($value);
        $this->value = $value;
    }

    public function validate(string $value): void
    {
        if ($value === '') {
            throw new InvalidArgumentException('UserName is required.');
        }

        if (mb_strlen($value) > self::MAX_LENGTH) {
            throw new InvalidArgumentException('UserName must be less than ');
        }
    }
}
```

上記のように、`UserName`クラスは以下の2つのバリデーションルールを持ちます。

1. コンストラクタで渡された値が空文字ではないこと
2. コンストラクタで渡された値の文字列長が最大文字数未満であること

これらのルールに違反した場合はどちらもLaravelの組み込み例外クラスである`InvalidArgumentException`を投げます。  
ただし、エラーメッセージはそれぞれのバリデーションルールに沿ったものを持ちます。

## Service(UseCases)

次に考えるのはServiceです。  

アプリケーションにおけるユースケースを表現するのがこのServiceになります。  
コード例では基本的にDIしてプロパティを保持します。

DDDにおけるアプリケーションサービスと似たような責務を持ちます。

DDDにおけるアプリケーションサービスの例ではよくCRUDのメソッド等がここに記載されていたりしますが、単一責任の原則に従うためにすべて別クラスに分割すべきだと思っています。

今回はアプリケーションサービスとは違い、1ユースケースのみを表現したクラスを用意します。

このクラスでは使用しているEntity,ValueObjectで発生した例外をハンドリングする必要が出てきます。

ここでの解決策としてはServiceクラス特有の例外クラスを作成します。  
自作例外クラスではなく共通で使用する例外クラスに引数にメッセージ等を渡して例外を生成してもいいと思いますが、個別にクラスを作ってしまうことでより表現力が増すことになります。

```php
class CreateUser implements CreateUserInterface
{
    private UserRepositoryInterface $repository;
    private UserFactoryInterface $factory;

    public function __construct(
        UserRepositoryInterface $repository,
        UserFactoryInterface $factory
    )
    {
        $this->repository = $repository;
        $this->factory = $factory;
    }

    public function process(
        CreateUserInputPort $input,
        CreateUserOutputPort $output
    ): CreateUserOutputPort
    {
        try {
        $user = $this->factory(
            $input->name(),
            $input->email()
        );
            $repository->save($user);
        } catch(Throwable $e){
            throw new CreateUserFailedSaveException($e);
        }
        return $output->output($user);
    }
}
```

自作例外クラスでは、以下のようにエラーメッセージの生成や例外に関する情報を隠蔽します。

```php
use RuntimeException;

class CreateUserFailedSaveException extends RuntimeException
{
    public function __construct(
        string $message = 'Failed to save when creating user.',
        int $code = 500,
        Throwable $previous = null
    ) {
        parent::__construct($message, $code, $previous);
    }
}
```

例外クラスを作成すればこのように、例外固有のメッセージやステータスコード等を持つことができます。  

ここで、エラーメッセージはかっこよく英語で書いていますね(あとで困ることになりますのでActionの説明まで覚えておいてください)

`CreateUser`では、InputPortとOutputPortは単純なDTOとして使用しています。

クリーンアーキテクチャにおいて必ずしもDTOであるとは限らないと考えているんですが、Web APIを返すバックエンドの実装においてはJSONを返すのでPresenterというよりはControllerが最終的に出力の役割を果たします。  
本来の文脈では`Controller => ユーザーの入力をUseCases層に渡す`、`Presenter => UseCases層からの出力を表示に適した形に加工する`という役割だと思います。

InputPort,OutputPortはUseCases層なのでHTTP通信であることやJSONといったWeb特有の文脈に依存させないために、ただ入力と出力のデータを受け渡すためのDTOとして実装すべきだと考えています。

Web以外のアーキテクチャにおいてはOutputPortを用いてPresenterが出力の責務(および出力のためのデータ加工)を全うすることになるのかなと思います(このパターンの実装経験が無いので想像になりますが、例えばコンソールアプリケーションのように標準出力を用いる場合はWebアプリケーションと異なる形になると思います)。

Factoryについては下記の様な実装になるかと思います。
```php
use Symfony\Component\Uid\Ulid;

/**
 * Factoryパターンの実装です。
 *
 * エンティティを生成する流れを隠蔽します。
 * 今回の場合はIDの採番に関するロジックを隠蔽します。
 */
class UserFactory implements UserFactoryInterface
{
    public function createUser(
        UserName $userName,
        UserEmail $userEmail
    ): User
    {
        return new User(
            Ulid::generate(), // ここではSymfonyを用いてULIDを生成することを想定してます。
            $userName,
            $userEmail
        );
    }
}
```

Repositoryは永続化を隠蔽しますが今回は省略します。

## Action(Controllers) ~ json形式でレスポンスを返すまで

Webのバックエンドにおいて、リクエストの内容を`Service`に受け渡す役割を担うのがこのActionになります。  
入力を受け取るので、HTTP通信であることはこの層で初めて意識することになると思います。  
そのため、この層ではHTTPExceptionとして例外を扱い、最終的にHTTPエラーレスポンスとして返します。  
レスポンスの形式については[RFC7807](https://datatracker.ietf.org/doc/html/rfc7807)で定義されているのでこれに沿う形のJSONを返してあげるのがベストプラクティスかなと思います。

実装例を書いてみます。

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Response;
use InvalidArgumentException;

class CreateUserAction
{
    private CreateUserInterface $createUser;

    public function __construct(
        CreateUserInterface $createUser
    ) {
        $this->createUser = $createUser;
    }

    public function __invoke(CreateUserRequest $request)
    {
        try {
            try {
                $input = new CreateUserInput(
                    new UserName($request->name()),
                    new UserEmail($request->email())
                );
            } catch (InvalidArgumentException $e){
                throw new BadRequestException($e->getMessage());
            }

            DB::transaction();

            try {
                $createdUser = $this->createAdminUser->process($input);
                DB::commit();
            } catch (CreateUserFailedSaveException){
                DB::rollback();
                throw new InternalServerErrorException($e->getMessage());
            }
        } catch (BadRequestException $e){
            // JSONレスポンスの作成
            return Response::json([
                'title' => $e->title(),
                'status' => $e->code()
                'detail' => $e->detail(),
                ]);
        } catch (InternalServerErrorException){
            // Laravelの場合は500系エラーをHandler等で共通して処理したり記録したりすることができる
            throw Response::json([
                'title' => $e->title(),
                'status' => $e->code()
                'detail' => $e->detail(),
            ]);
        }
        return Response::json($createdUser->toArray());
    }
}
```

Action層より内側で発生した例外はエラーメッセージを持った状態でcatchできるので例えば、下記のように記載することもできます。
```php
try{
// 処理内容
} catch (CreateUserFailedSaveException){
    throw new InternalServerErrorException($e->getMessage());
}
```

ですが、この方法では内部で発生した例外メッセージ(今回は英語で記載)をそのままJSONとして返したくない場合に対応できません。  
内部のエラーメッセージを変更することは、内部実装がプレゼンテーションのための知識を持つことになるため避けたい、でも例外によってエラーメッセージを分けて表示したいといったケースです。

最初の例のように、エラーメッセージの内容をActionで決めてしまうのが現状良いかなと考えています。

また、全く同じことが正常系の場合にも言えます。

```php
return Response::json($createdUser->toArray());
```

この部分ではエンティティの`toArray`メソッドの実装によって出力の形式が変わってしまいます。もちろん内容が変更された場合は出力も変更されるべきなのですが、ドメインロジックの配列に格納される順番に出力が依存してしまっているため切り離したいところです。

これらの問題は、クリーンアーキテクチャにおいては依存の方向性が内側に向いているため明確な違反ではないと考えていますが、切り離したほうがより綺麗な気がします。

![クリーンアーキテクチャ](/images/clean_arch.png)

改めてクリーンアーキテクチャの図を見てみると、`Presenters`という層が存在します。

先程のコード例では、Webという仕組みを意識した上で`Action`に`Controllers = 入力`という役割と`Presenters = 出力`という役割を持っている状態になります。

より丁寧に書くのであれば、`Presenter`となる層を用意するといいかと考えています。

HTTPのための例外である`BadRequestException`や`InternalServerErrorException`から生成していた処理や、エンティティが持っていた`toArray()`といったメソッドをPresenterに実装する事ができます。

まずは例外処理の方から考えてみます。

```php
throw Response::json([
    'title' => $e->title(),
    'status' => $e->status()
    'detail' => $e->getMessage(),
]);
```

だと例外のメッセージがそのまま外に漏洩するので

```php
throw Response::json([
    'title' => $e->title(),
    'status' => $e->status()
    'detail' => '任意のメッセージ',
]);
```

のようになります。ここで、タイトルやステータスは例外クラスごとに決まるはずなのでHTTPExceptionのインスタンスを生成するタイミングで`detail`を渡してしまえばいいと思います。

```php
try {
    $input = new CreateUserInput(
        new UserName($request->name()),
        new UserEmail($request->email())
    );
} catch (InvalidArgumentException $e){
    throw new BadRequestException('不正なリクエストです.');
}
```

HTTP用の例外クラスの実装例
```php
use RuntimeException;
use Throwable;

class BadRequestException extends RuntimeException implements HttpExceptionInterface
{
    private string $detail;
    private string $title;
    private int $status;
    private Throwable $previous;

    public function __construct(
        string $detail,
        string $title = 'Bad Request',
        int $status = 400,
        Throwable $previous
    ){
        parent::__construct($title, $status, $previous);
        $this->detail = $detail;
        $this->title = $title;
        $this->status = $status;
        $this->previous = $previous;
    }

    public function title(): string
    {
        return $this->title;
    }

    public function status(): int
    {
        return $this->status;
    }

    public function detail(): string
    {
        return $this->detail;
    }
}
```

例外クラスから最終的にjsonに変換する必要があるのでそのためにpresenterを使用します。もともとの例ではファサードを使用していて十分スッキリ書けてはいるので責務の分離が主目的になります。

```php
use Illuminate\Support\Facades\Response;

class ExceptionPresenter
{
    public static function makeJsonResponse(HttpExceptionInterface $e): JsonResponse
    {
        return Response::json([
            'title' => $e->title(),
            'status' => $e->code()
            'detail' => $e->detail(),
        ]);
    }
}
```

エンティティの`toArray()`がそのまま外に漏洩することを防ぐためにpresenterを使用する場合も同様です。

```php
use Illuminate\Support\Facades\Response;

class UserPresenter
{
    public static function makeJsonResponse(User $user); JsonResponse
    {
        return Response::json([
            'userId' => (string)$user->userId,
            'userName' => (string)$user->userName,
            'userEmail' => (string)$user->userEmail
        ]);
    }
}
```

以上を踏まえて最終的なActionは以下のような形になります。  
ここまでやればクリーンアーキテクチャのかなりの部分をコードで表現できたんじゃないでしょうか。  

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Response;
use InvalidArgumentException;

class CreateUserAction
{
    private CreateUserInterface $createUser;

    public function __construct(
        CreateUserInterface $createUser
    ) {
        $this->createUser = $createUser;
    }

    public function __invoke(CreateUserRequest $request)
    {
        try {
            try {
                $input = new CreateUserInput(
                    new UserName($request->name()),
                    new UserEmail($request->email())
                );
            } catch (InvalidArgumentException $e){
                throw new BadRequestException('不正なリクエストです.');
            }

            DB::transaction();

            try {
                $createdUser = $this->createAdminUser->process($input);
                DB::commit();
            } catch (CreateUserFailedSaveException){
                DB::rollback();
                throw new InternalServerErrorException('サーバーエラーが発生しました.');
            }
        } catch (BadRequestException $e){
            // JSONレスポンスの作成
            return ExceptionPresenter::makeJsonResponse($e);
        } catch (InternalServerErrorException){
            // Laravelの場合は500系エラーをHandler等で共通して処理したり記録したりすることができる
            throw ExceptionPresenter::makeJsonResponse($e);
        }
        return UserPresenter::makeJsonResponse($createdUser);
    }
}
```

## バリデーションについて

Webアプリケーションを構築する際に、フレームワークで投げられる例外から、クリーンアーキテクチャ、ドメイン駆動設計に基づいて分割された様々なオブジェクトから例外がスローされます。

これについての私の意見は以下のブログの内容とほぼ同じであるため、説明を譲ります。

https://ikenox.info/blog/validation-in-clean-arch/

それぞれの層が、それぞれの関心事、責務に基づいた例外をスローし、最終的に今回記事にしたような形でレスポンスに変換できれば良いと思っています。


## まとめ

- 基本的には型で例外をハンドリングするべきなので自作型を作ろう

型にしてしまうことで例外特有の情報をカプセル化する事ができます。PHPにおいては`try...catch`の際に型を使用できるので更に扱いやすくなります。

- ドメイン層にHttpExceptionを書くな

ドメイン層で投げられる例外はHTTPのことを意識するべきではないので、HttpExceptionはドメイン層に書かないようにすべきです。きちんと区別して使い分けましょう

- ドメイン層はプレゼンテーション層を意識しないため、エラーメッセージをAction層でコントロールするといいかも

いわゆるPresenterと呼ばれるクラスや処理を用意してAction層でそれらの処理を呼び出してコントロールするとドメイン層の情報をそのまま外に漏らすことがなくなります。
ただ、この点についてはドメイン層に依存してもクリーンアーキテクチャの本質である`依存の方向性を内側に向ける`というルールに反しているわけではないので責務の分離のための自己満足の面も多分に含んでいます

## 所感

普段業務でクリーンアーキテクチャを意識しながらコードを書いていても実装するたびに新しい疑問が湧いてきてどうすれば良いんだろう?と手が止まってしまうことが多々あります。  
今回はエラーハンドリングに関しての現時点の考えをまとめることができたので自身の考えの変化をwatchしてみたいと思います。  
今後もフレームワークのキャッチアップや手を動かして何かを作っていく中で自分なりに考えがまとまったら定期的にアウトプットしたいと思います。