---
title: Laravelを普段使っているがRuby on Railsに入門してみる
tags: [Ruby on Rails]
createDate: 2023-03-12
updateDate: 2023-03-12
slug: ror-01
draft: true
---

## 20分ではじめるRubyに目を通す

<https://www.ruby-lang.org/ja/documentation/quickstart/>

### メモ

全てがオブジェクト
`do...end`でクロージャがかける

多重継承はできないがモジュールの概念があり、メソッドを自由に受け取ることができる。

変数
- `var`はローカル変数
- `@var`はインスタンス変数
- `$var`はグローバル変数

例外処理ができる
GCもある

関数定義
例:

```ruby
def hi
puts "Hello World"
end
```
`def`で関数を定義して`end`までの間に処理を書く
引数を持たない場合は`()`を省略することができる

文字列に何かを挿入したい場合は`#{変数名}`のように書く

class定義は普通に`class クラス名`で最後は`end`

`クラス名.new()`でオブジェクトの生成

どんなメソッドが定義されているかは

```ruby
クラス名.instance_methods(); #　親や祖先のメソッドも含む
クラス名.instance_methods(false); #　親や祖先のメソッドを含まない
```


## ダックタイピングとは

静的型付け言語のほうが馴染みが強い(PHPもほぼ静的型付けしてるし)ので馴染みがなかった
コンパイル時に型の検査をするのではなく、実行時に実行する。
オブジェクトに何ができるかはクラスではなく実行時のオブジェクトそのものが決定するという考え方


>"If it walks like a duck and quacks like a duck, it must be a duck"
>（もしもそれがアヒルのように歩き、アヒルのように鳴くのなら、それはアヒルに違いない）

Rubyは一般的なクラス継承もできるので継承によるポリモーフィズムも利用できるが、ダックタイピングを使えば継承が不要であり型による制約に縛られることなく簡素なコードで実現できる
制限がないということは乱用に繋がる
->この辺が設計的によわいかも。早く作るだけならまだしも頑張ろうとするときつそう

Rubyにおいては`respond_to?`メソッドによってオブジェクトがメソッドを持つかどうかをチェックできる
https://docs.ruby-lang.org/ja/latest/method/Object/i/respond_to=3f.html


40分くらいで寄り道しながら目を通したので
PHPからRubyへも読む(10分かからないくらい)
https://www.ruby-lang.org/ja/documentation/ruby-from-other-languages/to-ruby-from-php/

Rubyはオブジェクトを意識しておけばある程度はなんとかなりそう

Railsの基本思想をつかむ

https://railsdoc.com/

`rails new アプリケーション名`
というコマンドだけでアプリケーションを作れる
railsはフルスタックなフレームワークだけどAPIのみを作成することもできる
作成時に`-api`オプションをつけるだけ。

```
$ rails generate scaffold 名前 [カラム名:型[:index]..] [オプション]
```

でアプリケーションの基本的な機能を作れる


基本的にrailsのテンプレートにしたがうのであればコマンドで何もかも作れる
Laravelもこの辺りの機能はあるのである程度早く作ることはできるが設計上あまり利用してこなかった
押さえておいた方が良さそうなのは

- ORM(ActiveRecord)
- migrate,seed(Laravelとほぼ同じか?)
- Action Mailbox
- Active Storageとは?
- railsにおけるテスト(システムテストという概念があるっぽい。結合テストを指している?)

個人開発でお題が簡単な場合に使ってみてもいいかも。早そう
fat controllerは避けられないが

テストについて
おそらく今回のセッションの件は保存してるセッションさえ消せれば良さそう。

残タスクはフロント側のcookieなりに残っているほうを消すこと
あとはエンドポイントのテストが問題。

https://railsguides.jp/testing.html

RAILS_ENV=test環境でテストが実行される

`require "test_helper"`でデフォルトのテストヘルパーが読み込まれる。

`test`マクロがあるので使用すると読みやすいテスト名でかける

```ruby
test "the truth" do
    assert true
end
```

```ruby
def test_the_truth
    assert true
end
```

テストの実行順序はランダム
`Minitest`という名前のテスティングライブラリらしい

```
getメソッドはWebリクエストを開始し、結果を@responseとして返します。このメソッドには以下の6つの引数を渡すことができます。

リクエストするコントローラアクションのURI。 これは文字列ヘルパーかルーティングヘルパーの形を取る（articles_urlなど）。
params: アクションに渡すリクエストパラメータのハッシュ（クエリの文字列パラメータまたはarticle変数など）。
headers: リクエストで渡されるヘッダーの設定に用いる。
env: リクエストの環境を必要に応じてカスタマイズするのに用いる。
xhr: リクエストがAjaxかどうかを指定する。Ajaxの場合はtrueを設定。
as: 別のcontent typeでエンコードされているリクエストに用いる。デフォルトで:jsonをサポート。
```

https://railstutorial.jp/chapters/log_in_log_out?version=4.2

### `:(コロン)`について

```ruby
:hoge
```
文字列の先頭のコロンで`シンボル`を表現する   
<https://zenn.dev/kanoe/articles/352d78902c83e168db66>

ハッシュのデータを扱う時に見かけることが多い

シンボルはオブジェクトで、内部実装でメソッド名や変数名、クラス名などの名前を整数で管理している
直接文字列として処理するよりも速度面で有利

https://docs.ruby-lang.org/ja/2.6.0/class/Symbol.html


## 名前付きルート

`sessions_path`のようなもの

https://railsguides.jp/routing.html

リソースフルなルーティングを作成するとアプリケーションで多くのヘルパーが利用できるようになる
`sessions_path`は`session`というリソースにたいしてのルート設定なので`/sessions`を返す

## メソッド名の後ろにつける`!`について
メソッドの最後に！をつけることにより、レコードの作成などに失敗した際の挙動をかえることができる。 →　例外処理を書く際に利用できる。

## 所感

静的型付けで書きたい気持ちは結構強い
チェック自体もそうだがコードの表現力が結構変わってくる
設計を頑張るとなると抽象クラスがないのが結構きついかも
雰囲気はPHPよりjsの方が近いかもしれない


https://qiita.com/tobita0000/items/866de191635e6d74e392