---
title: LaravelのEagerLoadについて
tags: [Laravel]
createDate: 2022-10-23
updateDate: 2022-10-23
slug: laravel-eagerload
---

## EagerLoadとは

- 「Eager」は「熱心」という意味
- ORMにおけるN+1問題を解決するために使用される
- 先に取得したModelに対して事前にリレーションを取得する仕組み
- Eloquentの場合はプロパティとしてリレーションにアクセスする場合に行われる。

## 例
まずは[公式サイト](https://readouble.com/laravel/7.x/ja/eloquent-relationships.html#eager-loading)にある例をもとに理解を進めます。

### N + 1問題とは
ORMによるリレーションを扱う際には `N + 1問題`が発生することがあります。
EagerLoadはその解決策ですがまずは`N + 1問題`について公式の例を確認します。

```PHP
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * この本を書いた著者を取得
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}
```

```PHP
$books = App\Book::all(); // 1. すべての本を取得

// 2. 著者をそれぞれの本について取得
foreach ($books as $book) {
    echo $book->author->name;
}
```

上記ループでは

1. まずテーブルからすべての本を取得するために1回クエリが発行される。
2. 著者をそれぞれの本について取得。本の数だけクエリが発行される。

という流れでクエリが発行されます。つまり、25冊の本について情報を取得する(N=25)場合、25回 + 全体のデータ取得の1回で合計26回のクエリが発行されることになります。

そもそもこの問題の原因は、Eloquentの仕組みにあります。
Eloquentは、**リレーションにアクセスする度にクエリを発行する**という仕組みです。
このクエリの発行回数を抑えるために、事前にリレーション情報を含めた状態の親モデルを取得するEagerLoadという仕組みを使用する事ができます。



### EagerLoad
```PHP
$books = App\Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

上記のように`with`メソッドを使用することでクエリの発行回数を抑える事ができます。
発行されるクエリは以下のようになります。

```SQL
select  * from books

select * from authors where id in (1,2,3,4,5,...)
```

複数のリレーションに対するEagerLoad
```PHP
$books = App\Book::with(['author', 'publisher'])->get();
```
ネストしたリレーションに対するEagerLoad
```PHP
$books = App\Book::with('author.contacts')->get();
```
特定カラムに対してのEagerLoad
```PHP
$users = App\Book::with('author:id,name')->get();
```
EagerLoad時に制約を追加する
```PHP
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
```

遅延EagerLoading
すでに親のモデルを取得したあとにリレーションをEagerLoadしたい場合に利用します。   
どのリレーションをロードするか動的に決定したい場合に便利です。
```PHP
$books = App\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```
`loadMissing`メソッドを使用すると、リレーションをまだロードしていない場合のみロードする事が可能です。
```PHP
public function format(Book $book)
{
    $book->loadMissing('author');

    return [
        'name' => $book->name,
        'author' => $book->author->name
    ];
}
```

## Eloquentにおける動的プロパティについて
例えば以下のような1対多野リレーションがあったとします。
```PHP
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * ブログポストのコメントを取得
     */
    public function comments()
    {
        return $this->hasMany('App\Comment');
    }
}
```
この場合に
```PHP
$post = Post::find(1);
$comments = $post->comments;
```
のように取得すると思います。   
これは`comments()`メソッドではなく`comments`という動的プロパティを返しているという点に注意が必要です。   
`comments`は`Comment`モデルのインスタンスのコレクションを返します。   

反対に、`comments()`メソッドは`HasMany`オブジェクトを返します。

### 動的プロパティとリレーションメソッドの違い
モデルのインスタンスの中身を確認すると、インスタンスのプロパティには`attributes`のほかに、`relations`というものがあります。   
個々にはkeyがリレーションメソッド名、valueがリレーション先のモデルインスタンスのコレクションとした連想配列が格納されます。

`HasMany`, `HasOne`などのリレーションメソッドではクエリビルダを使用する事ができます。   

EagerLoadにおいては動的プロパティを利用する必要があります。   
動的プロパティは「遅延ロード」されます。遅延ロードはアクセスされたときにだけリレーションのデータをロードするという仕組みです。   
リレーションメソッドを使用してしまうと、EagerLoadしても毎回クエリを発行してデータを取りに行ってしまうことになるため、効果が無いことに注意が必要です。

まとめると、

- 動的プロパティを使用することでアクセスされたときにクエリを発行してデータを取ってくる
- `with`でリレーション先のデータを`relasions`プロパティに格納しておく
- `relations`プロパティにデータが存在すれば動的プロパティはクエリを発行せずにそのデータを使用してくれる



## 所感
今回はEagerLoadについて調査しました。   
Eloquentのリレーションの仕組みやロードのタイミングについて理解が深まったと思います。   
EagerLoadという単語自体は聞いたことありましたが、使用しない場合と使用した場合の差や、デフォルト状態のEloquentのデータを取得する仕組みについて理解できました。   
Eloquentを利用する際には仕組みを理解しておくことで実行時間の予測がある程度できるようになるため、クエリビルダを利用するなど別の手段との比較がしやすくなるなと感じています。   

## 参考
https://katsusand.dev/posts/laravel-eager-load/   
https://readouble.com/laravel/7.x/ja/eloquent-relationships.html#eager-loading   
https://qiita.com/179Bell/items/7b4f991816f63946e738    
https://qiita.com/mpyw/items/ed058e2d679a672c3ba7   