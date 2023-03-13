---
title: Mockeryの基本的な使い方
tags: [test]
createDate: 2022-07-22
updateDate: 2022-07-22
slug: mockery
---

## はじめに
前回の記事でユニットテストについて書きました。
今回は前回にも使用したMockeryというフレームワークについて少し掘り下げてみたいと思います。

## 公式

https://readouble.com/mockery/1.0/ja/index.html

公式にもあるように、他のテストフレームワークと合わせてユニットテストで使用します。

PHPUnit公式

https://phpunit.readthedocs.io/ja/latest/index.html

## よく使う機能(入門編)

### モックの作成

Mockeryにおいて、スタブとモックは同じものが生成されます。   
スタブは指定した結果を返すだけですが、モックは期待しているメソッド呼び出しのエクスペクションを指定できます。   
以下すべてモックと呼びますが、スタブとしても使用できます。   
モックはスタブの機能を含んでいる形です。   
また、他のテストダブル(代替物)に**スパイ**というものがあります。   

公式にて推奨されるモックの作成方法は、以下のように具象クラス名を指定する方法です。   
この方法で生成されたモックオブジェクトは継承により、`MyClass`という型を保ちます。   

```php
$mock = \Mockery::mock('MyClass');
```

推奨されるのは上記ですが、モックオブジェクトは具象クラス、抽象クラス、そしてインターフェイスでもベースに指定することができます。   
タイプヒントのために特定の方をモックオブジェクトに継承させたい場合に有用です。   

```php
$mock = \Mockery::mock('MyInterface');
```

このモックオブジェクトは`MyInterface`型を実装しています。

唯一作成できないモックは`final`クラスですが、こちらについてはパーシャル(部分)モックという仕組みが用意されています。   
こちらについては後ほど述べます。   

Mockeryは一つのクラスで複数のインターフェイスを実装するクラスに基づいたモックも作成できます。   
僕が単体テストを書く際にもこの形で記述しています。   

```php
$mock = \Mockery::mock('MyClass, MyInterface, OtherInterface');
```
単体テストを抽象に対して書きたいという気持ちはあるのでインターフェイスのみを指定してモックを生成してもいいかもしれません。   
ただ、公式では

>Note: リストの最初の項目であるクラス名は必須ではありませんが、指定したほうが読みやすくフレンドリーでしょう。

と記載があるので具象クラスを指定して記述するようにしています。

抽象クラスを指定して、メソッドの期待している動作を書いていないまま呼び出したらどうなるんでしょう？
```
Received モッククラス名::メソッド名(), but no expectations were specified
```

というようなエラーが出てテストに失敗します。   
モック以外を使用したくない場合等Interfaceを指定するとよさそうな場面もありそうです。   

### モックに期待動作を設定する
Mockeryではモックの動作に期待する内容を簡単に指定することができます。

```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('name_of_method');
```

最も基本的な期待動作の指定はこのようにかけます。   
`shouldReceive`を書くことでそのメソッドが呼び出されるのを期待することをテストダブルに伝えることができます。

次に戻り値の指定です。
```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('name_of_method')
    ->andReturn($value);
```

`andReturn`を使用することでどのような戻り値が返されるのかを指定できます。   
これで期待した値を返すよう指定できるのでスタブとして使用することができるようになりました。    

### 複雑な返り値の指定
他に考えられるパターンとして、テスト中に複数回モックメソッドが呼び出され、呼び出されるたびに異なる返り値を返したい場合です。
```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('name_of_method')
    ->andReturn($value1, $value2, ...)
```
これは`andReturn`に複数指定するだけで順番に返されます。

`andReturnValues([$value1, $value2, ...])`のように`andReturnValues`を使用すれば配列の形で指定することもできます。   
いずれの場合も指定した返り値の数より大きい回数呼び出された場合は最後の要素が返されます。   

### 呼び出し回数のテスト

```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('name_of_method')
    ->times($n);
```

このようにモック化したメソッドが何回呼び出されることを期待するかを指定できます。   
指定した回数と実際の呼び出し回数が一致しなかった場合は`\Mockery\Expectation\InvalidCountException`が投げられます。   
また、`times`以外にも1回であれば`once`、2回であれば`twice`、呼び出されないことを指定する場合は`never`等が指定できます。   

最低n回呼び出したい、もしくは最高実行回数はn回という指定をしたい場合は`atLeast`や`atMost`が使用できます。   

### 例外のテスト
これまで動作の指定について調べましたが、異常系のテストの場合はモック化したオブジェクトで例外を投げたいパターンが考えられます。

```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('name_of_method')
    ->andThrow(Exception);
```
これを使用することでモック化しない場合と比べて簡単にエラーハンドリングのテストを書くことができます。


### モックに渡された引数を確認する
公式:   
https://readouble.com/mockery/1.0/ja/argument_validation.html

モックに期待する動作として、モックに渡された引数の中身を確認したい場合が考えられます。
これは、以下のように書くことができます。
```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('foo')
    ->with(1):
```

プリミティブ型の場合は`with()`を使用することで簡単に検証できます。
ただ、
>このようなケースでは、Mockeryはまず引数の比較に===（厳密な比較）演算子を使用します。引数がプリミティブで、厳密な比較で不一致の場合、Mockeryは==（緩やかな比較）演算子をフォールバックとして使用します。

とあるので厳密な比較のみ行いたい場合は注意が必要です。


```php
$object = new stdClass();
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive("foo")
    ->with($object);
```
オブジェクトの場合は厳密な比較を行うので、全く同じオブジェクトのみが一致します。

上記以外で困るのがオブジェクトのプロパティに対して検証したい場合です。   
こちらもMockeryで対応する方法が用意されています。
```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive("foo")
    ->with(\Mockery::on(closure));
```

具体的に、PHPUnitと組み合わせて使うと
```php
$mock->shouldReceive('foo')
    ->with(Mockery::on(function (Argument $arg) use ($expect) {
        $this->assertSame($expect, $arg);
        return true;
    }));
```
こんな感じで書けます。   
ここでの`$arg`はモックに渡された引数です。   
`$expect`はテストドライバ内で用意しておいてクロージャで使用します。   
クロージャの返却値が`true`であればその引数はエクスペクションと一致したと判断されるのでこのような書き方が可能となります。   


### モックの使い方(発展編)

### パーシャルモック
あるオブジェクトのいくつかのメソッドのみをモックし、残りは実際のメソッド通りに動作させることができます。

https://readouble.com/mockery/1.0/ja/partial_mocks.html

```php
$mock = \Mockery::mock('MyClass')->makePartial();
```
基本的には`makePartial`を使用することでパーシャルモックを生成できます。
コンストラクタに引数を指定したい場合は、第二引数に渡すだけです。
```php
$mock = \Mockery::mock('MyClass', [$arg1, $arg2])->makePartial();
```

ただし、finalクラスやfinalなメソッドをモックしたい場合はプロキシパーシャルモックを使用する必要があります。
```php
$mock = \Mockery::mock(new MyClass);
```
これは呼び出しを横取りして期待した動作に合わないメソッドは引数に渡したオブジェクトへ引き渡します。   
ただ、モックしているクラスのタイプヒントのチェックは失敗します。   
`final`なクラスを拡張することができないからです。   

### staticなメソッド
このあたりから少し特殊なテストになります。
https://readouble.com/mockery/1.0/ja/public_static_properties.html

staticなメソッドは実際のオブジェクト上で呼び出されないため、これまで紹介した方法ではモック化できません。   
この問題の解決のために、エイリアスモックという仕組みが用意されています。   
これを使用することでstaticメソッド呼び出しを横取りしてエクスペクションを追加できるようになります。   

```php
$mock = \Mockery::mock('alias:MyClass');
```

注意が必要なのは、このような形の単体テストを複数書く場合です。   
これは結構あり得る話かと思います。   

>２つ以上のテスト間で、エイリアス／インスタンスモックを使用すると、同じ名前の２つのクラスは持てないため、fatalエラーが発生します。
これを防ぐには、この種のテストは、独立したPHPプロセスで実行してください。PHPUnitとPHPTで、サポートされています。

公式でも上記のように書かれています。   
これの対策にはPHPUnitで用意されているアノテーションを使用します。   
PHPUnit: https://phpunit.de/manual/6.5/ja/appendixes.annotations.html#appendixes.annotations.preserveGlobalState

```php
/**
 * @runInSeparateProcess
 * @preserveGlobalState disabled
 */
public function testfunction(): void
{
}
```

### publicプロパティのモック
モック化したオブジェクトのpublicプロパティへ特定の値をセットしたい場合は
```php
$mock = \Mockery::mock('MyClass');
$mock->shouldReceive('name_of_method')
    ->andSet($property, $value);
```


### スパイの作成
Mockeryで作成できる、テストダブルにはスタブ/モックともう一つ、スパイと呼ばれるものを作成できます。   
スパイはテストダブルに対して行われた呼び出しをテスト対象の呼び出し後に検査することができます。   
モックの場合は呼び出す前に期待する動作を指定する必要がありました。   

```php
$spy = \Mockery::spy('MyClass, MyInterface, OtherInterface');
```

ただ、スパイの場合はメソッド実行の戻り値を指定したりすることはできません。

https://readouble.com/mockery/1.0/ja/spies.html

スパイを使用することのメリットは、テストコードがより直感的になることです。   
モックの場合は期待する動作を実際の呼び出しよりも前に記述する必要がありました。   
スパイの場合は呼び出し後に   

```php
$spy->shouldHaveReceived('foo')
    ->with('bar');
```

のような形で記述できます。

ただ、モックと比べると機能は限定的で、個人的にはスパイよりモックを使用すべきだと考えています。


## 他のモックフレームワークとの比較
以上の内容を押さえておけば大抵のケースのユニットテストは書けると思います。   
他にも公式ドキュメントにトリック的な例も載っていたりするので参照してください。

ところで、他にPHPではどのようなテストフレームワークがあり、Mockeryとどのような点で違うのかについて調査してみます。

今回の比較対象として、

- PHPUnitで使用できるモック機能
- Mockery

の2つを比較してみます。
ほかにもPhake等フレームワークはあるのですが、Laravelを使用する場合は選択肢はこの二つに絞られそうです。

PHPUnit公式 テストダブル   
https://phpunit.readthedocs.io/ja/latest/test-doubles.html

Mockery   
https://readouble.com/mockery/1.0/ja/index.html

参考:   
https://rimuru.lunanet.gr.jp/notes/post/2953/

PHPUnit   
Example 8.12抜粋
```php
$observer->expects($this->once())
            ->method('update')
            ->with($this->equalTo('something'));
```
この部分でモックの期待動作の設定を行なっていますが、ほぼMockeryと同じような形で指定できます。   
ただ、公式ドキュメントを見る限り、これ以上複雑な例になるとMockeryの方が簡潔に書くことができます。   
また、参考サイトにもありますが、メソッド呼び出し順序についてはPHPUnitでは検証できないようです。   

## 所感
Laravelを使用する場合はMockryを使用するといい感じにテストを書けそうです。   
また、簡単な返り値のテストのみであればPHPUnitから使ってみるのもいいかもしれません。   
ただ、テスト自体が泥臭く、いろんなコードをテストコードに書きがちです。   
その中でもテストコードは仕様を表現するべきであり、できるだけ直感的に把握できるべきだと考えています。   
そう考えるとMockeryは簡潔に書くことのできるフレームワークだと感じました。   
Laravelを使用している場合はインストールされていると思いますので簡単に使えることもメリットです。   

いいテストを書けるようになると、テストのしづらいコードに気づくことができるようになり、そういったコードは設計がいまいちだったりします。   
いい設計を身につけるためにもユニットテストは重要だなと感じていますし、いいテストが書けるようになるとリファクタリングが怖くなくなります。   
ドメインの変更にコードを追随させるためにも、いいテストが書けるよう工夫を続けていきたいと思います。
