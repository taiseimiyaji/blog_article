---
title: ORM(オブジェクト関係マッピング)について
tags: [DB]
createDate: 2022-09-28
updateDate: 2022-09-28
slug: orm
---


## はじめに
今回はORMについて調査したいと思います。   
また、よく比較されるクエリビルダについても調査し、その違いを理解したいと思います。

## ORMとは

ORMとは、Object-Relational Mappingの略です。

データベースとオブジェクト指向プログラミングの間のデータを変換するもの。   
つまり、データベースに対するデータの操作をオブジェクト指向型言語の操作方法で扱えるようにします。

オブジェクト指向...現実世界をモデル化したもの

関係データベース...検索やCRUD処理に最適化されたデータのためのモデル。

かなり大雑把な説明ですが、この二つの間には考え方の違いがあります。(インピーダンスミスマッチというらしいです)

ORMを用いれば直接SQLを書くことなく、オブジェクトのメソッドを用いてDB操作ができるようになります。   
例えば、createメソッドで新規作成するなど、オブジェクト指向のクラスのメソッドを用いてSQLを発行することができます。



## ORMのメリット

- SQLを直接書かなくてもいい
- オブジェクト指向言語で書ける
- テーブル同士のリレーションを表現できる
- データベースごとの言語の違いを吸収してくれる

## ORMのデメリット

- 各ORMライブラリの使い方を覚えないといけない
- 直接SQLを書くわけではないのでチューニング等の面に課題がある
- オブジェクトを返すので大抵の場合メモリ消費量が大きくなる

各言語ごとにORMのライブラリが用意されています。
業務で用いているLaravelの場合は代表的なものとして`Eloquent`という仕組みがあります。

## クエリビルダについて

PHPのメソッドを使用するような書き方でSQLを発行できる仕組み。


||書き方の例|取得できるもの|
|---|---|---|
|クエリビルダ|DB::table()|連想配列|
|ORM|Model::all()|モデルクラスのインスタンス|

クエリビルダの所属クラスは`Illuminate\Support\Facades\DB`、Eloquentの所属クラスは`Illuminate\Database\Eloquent\Model`となります。


**ちなみにEloquent形式で記述する際にもクエリビルダを使用することができます。**   
Eloquentのリレーションはクエリビルダとして機能するため、クエリビルダチェーンを使用できます。   
簡単に言うと、Eloquentにチェーンする形でクエリビルダを記述できます。   
これがEloquentとクエリビルダの違いをややこしくしている原因です。   


### クエリビルダのメリット

- SQLを書くようにメソッドをチェーンできるため、SQLの知識があれば用意に記載できる
- Eloquentと比較すると高速

### クエリビルダのデメリット

- 自分でテーブル結合をする必要がある
- Eloquentと比較するとSQLを意識して書く必要がある
- 記述量はEloquentと比較すると多い

例) `sample`というテーブルに対してクエリビルダを書く場合
```php
<?php
DB::table('sample')
    ->select()
    ->where(条件);
```
のような形で簡単にSQLライクに書けます。

## Eloquentにおけるリレーション

参考: https://readouble.com/laravel/8.x/ja/eloquent-relationships.html

テーブルのリレーションについて、Eloquentでのコード例を公式ドキュメントを参考にまとめます。

### 1対1(hasOne, belongTo)

`User`と`Phone`モデルが関連付けられている場合

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * ユーザーに関連している電話の取得
     */
    public function phone()
    {
        return $this->hasOne(Phone::class);
    }
}
```

上記のように`hasOne`メソッドを使用する。引数には関連モデルクラスの名前を渡す。

逆に、`Phone`モデル空`User`モデルへアクセスできるようにする場合は`belongTo`メソッドを使用することで`hasOne`関係の逆を定義できる。

あるモデルが多くの関連モデルを持つ場合、そのリレーションにおける「最新」または「最も古い」関連モデルを簡単に取得したい場合

```php
<?php
// 最新
$this->hasOne(Phone::class)->latestOfMany();
// 最も古い
$this->hasOne(Phone::class)->oldestOfMany();
```

デフォルトではソート可能なモデルの主キーに基づいて最新または最古の関連モデルを取得する。   
別のソート基準を使用したい場合は`ofMany`メソッドを使用する。   
引数には関連するモデルを検索する際にどの集約関数(`min`または`max`)を適用するかを指定する。   

```php
<?php
$this->hasOne(Order::class)->ofMany('price', 'max');
```


キー制約等は引数で指定できる。詳細は公式ドキュメントを参照

### 1対多(hasMany)

`hasOne`同様の使い方ができる`hasMany`メソッドを使用する。

`hasMany`の逆の所属関係を定義するには`belongsTo`メソッドを使用する。

### デフォルトモデル

`belongsTo`, `hasOne`, `hasOneThrough`, `morphOne`等リレーションを使用する場合は指定する関係がnullの場合に返すデフォルトモデルを定義できる。
一般的なNullオブジェクトパターンを実現できる。

### 多対多
- users...ユーザーテーブル
- roles...役割テーブル
- role_user...中間テーブル

`belongsToMany`メソッドを使用する。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * このユーザーに属する役割
     */
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}
```

リレーションを定義したあと、アクセスする際は`roles`動的リレーションプロパティを使用してユーザーの役割へアクセスできます。

```php
<?php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    //
}
```

逆の関係も同様に`belongsToMany`を使用します。
違いはUserモデルを参照することです。

また、中間テーブルのカラムの取得には`pivot`を使用する。

```php
<?php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```


### リレーションの存在のクエリ

モデルレコードを取得する際にリレーションの有無に基づいた結果を制約したい場合。   
例えば、コメントが少なくとも1つある全てのブログ投稿を取得する場合。   
`has`メソッドと`orHas`メソッドを使用する。   

```php
<?php
use App\Models\Post;

// コメントが少なくとも1つあるすべての投稿を取得
$posts = Post::has('comments')->get();
```

演算子を追加してクエリのカスタマイズする事もできる。

```php
<?php
// コメントが３つ以上あるすべての投稿を取得
$posts = Post::has('comments', '>=', 3)->get();
```

`whereHas`と`orWhereHas`を使用することでコメントの内容の検査などの追加のクエリ制約を定義できる。   

```php
<?php
use Illuminate\Database\Eloquent\Builder;

// code％と似ている単語を含むコメントが少なくとも１つある投稿を取得
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();

// code％と似ている単語を含むコメントが１０件以上ある投稿を取得
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
}, '>=', 10)->get();
```

以上のように、Eloquentを使用すると簡単にテーブルのリレーションを表現できます。   
さらに、リレーションを表現した上で追加でSQLを発行したい場合はクエリビルダも使用する事ができるため、簡単にDB操作を行うことができます。

## Eloquentと比較したクエリビルダのメリット

ここまで紹介したように、非常に便利なEloquentですが、クエリビルダの方が勝っている点もあります。   
はじめに紹介したメリットの中でも最も大きいのが`パフォーマンス`です。

Eloquentモデルのメソッドを使用する場合、メソッドの種類にもよりますが内部で都度SQLが実行される場合があります。   
それに対してクエリビルダの場合は明確にSQLの発行内容を指定できるためSQLの発行回数を抑えることができます。   
また、最終的にModelに詰められてデータが返されるEloquentと比較するとメモリの消費量も抑えることができます。   


## クエリビルダとEloquentの選択

選択基準としてはパフォーマンス(クエリビルダ)と扱いやすさ(Eloquent)のトレードオフになると思います。   
ただ、かなり強力な機能なのでEloquentを全く使用しない場合はLaravelを使用する理由の一つを犠牲にすることになるかなと思います。   

### CQRSを採用し、参照系はクエリビルダを用いる

クエリビルダとEloquentの選択において、基準としている考え方に`CQRS`があります。   
`コマンドクエリ責務分離`と呼ばれるもので、簡単に言うとサーバーサイドの機能を「コマンド(副作用があるもの)」と「クエリ(副作用のないもの)」で完全に分けてしまおうという考え方です。   

コマンドの場合は副作用、つまりデータストアへの変更が発生するため、型を用いた値の確認や整合性の確認のためのロジックを実行する必要があります。   
それに対してクエリの場合は副作用がないため、整合性等気にする必要性が薄いです。   
また、クエリの場合は正規化しているDBを用いる場合にデータの取得が煩雑になります。   
クエリはコマンドに比べると多く実行されるパターンが多いため、パフォーマンス面の負荷が大きくなります。

つまり、コマンドには複雑なビジネスロジックを表現したドメイン駆動を適用するが、クエリの場合は省略するという考え方です。

これについては以下のブログが参考になると考えています。

https://little-hands.hatenablog.com/entry/2019/12/02/cqrs

CQRSにもどの層で分けるべきかという話があって、これは段階的に分けることができるという考え方です。

- 共通のDBを利用し、コマンドとクエリで利用するモデルを分けて実装する
- DBをコマンド用とクエリ用で分ける
- イベントソーシングの考え方で実装する

イベントソーシングについては先程のブログの中で

>イベントソーシングとは、データ永続化をドメインオブジェクト(EntityやValueObject)の状態(ステート)をそのまま保存するのではなく、「ユーザーが登録された」「タスクが完了された」といった イベントそのものを永続化する というアーキテクチャです。

というように説明されています。

また、下記のスライドもわかりやすいと思います。

https://pages.awscloud.com/rs/112-TZM-766/images/DevAx_connect_jp_season1_day4_CQRS%26EventSourcing.pdf

## 参考

https://pages.awscloud.com/rs/112-TZM-766/images/DevAx_connect_jp_season1_day4_CQRS%26EventSourcing.pdf

https://little-hands.hatenablog.com/entry/2019/12/02/cqrs

https://readouble.com/laravel/8.x/ja/eloquent-relationships.html


## 所感
今回はORM(Eloquent)、クエリビルダについてそれぞれのメリットや違い等を理解するために調査してみました。   
次回以降、またパフォーマンス面での比較や内部で発行されるSQLについても調査してみたいと思います。   

