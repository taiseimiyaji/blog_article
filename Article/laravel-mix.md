---
title: Laravel Mixについて
tags: [Laravel]
createDate: 2022-04-21
updateDate: 2022-04-21
slug: laravel-mix
---
## 参考

https://readouble.com/laravel/6.x/ja/mix.html   
https://qiita.com/minato-naka/items/bfc3bbd9a388084e6f17   
https://qiita.com/kamykn/items/45fb4690ace32216ca25
https://qiita.com/minato-naka/items/0db285f4a3ba5adb6498
https://goworkship.com/magazine/how-to-webpack/
https://qiita.com/non_cal/items/a8fee0b7ad96e67713eb
https://qiita.com/righteous/items/e5448cb2e7e11ab7d477
https://udemy.benesse.co.jp/design/web-design/sass.html
http://blog.sakurachiro.com/2017/08/scss_sourcemap/


## はじめに
今回の記事はうまくまとまらなかったなと感じていますが、今時点の私の知識ではまとめきれないなと判断しました。
随時更新していきたいと考えています。
Laravelに初めて触った時に僕がつまづいたのはフォルダの多さでした。   
この辺りの層の分け方は設計云々も絡んでくると思うので今は触れませんが、
Laravelでは役割ごとにファイルを分けやすい仕組みが用意されていて、その一つが`Laravel Mix`です。   


## Laravel Mixとは

以下参考サイトより引用。
>Laravel Mixは多くの一般的なCSSとJavaScriptのプリプロセッサを使用し、Laravelアプリケーションために、構築過程をwebpackでスラスラと定義できるAPIを提供しています。

>webpackファイルをよりわかりやすく簡単に書けるように設定ファイルをラップしている。
フロントエンド開発では、webpackというものを使用して開発をするとメリットがあるのですが、設定ファイルが長い。この問題を解決するためのツールがLaravel Mixということらしいです。
ちなみに、Laravelでなくても使用できます。
<figure class="figure-image figure-image-fotolife" title="ざっくりしたイメージ図">[f:id:taisei_miyaji:20220421195840p:plain]<figcaption>ざっくりしたイメージ図</figcaption></figure>


## そもそもwebpackとは?

https://qiita.com/kamykn/items/45fb4690ace32216ca25
https://qiita.com/minato-naka/items/0db285f4a3ba5adb6498
https://goworkship.com/magazine/how-to-webpack/

ここが一番重要な部分で、webpackがどういうことをしているのかわかればMixはそれのラッパーのようなイメージなのかなと思っています。

webpackとは、主にjsをバンドルするためのモジュールバンドラ。
バンドラ...複数のファイルを1つにまとめて出力してくれるツールのこと。

ちなみに、js以外にもCSSや画像もバンドルできるらしい。

使用することで、

- ファイルを分割して開発できるため、開発効率や保守性の向上につながる。
- ファイルを1つにまとめることでリクエストの回数を削減し、パフォーマンスの向上につながる。

`webpack.config.js`に`entry:エントリーポイントファイル`と`output:バンドルされたファイル名と出力先パス`を書く。

この状態で`webpack`コマンドを実行すると`bundle.js`が出力される。
あとはこのファイルをhtmlファイルで読み込めば使える。

## Laravel Mixのインストール

Laravelをインストールした時点で`package.json`が存在しているはずなので、下記コマンドでインストールします。

```
npm install
```

この`package.json`はNodeの依存パッケージのための構成ファイルです。

Nodeとは、簡単にいうとサーバーサイドでJavaScriptを動かすための仕組みです。
Nodeについては以下を参照するとすんなり理解できるかなと思いました。

https://qiita.com/non_cal/items/a8fee0b7ad96e67713eb

PHPの場合は`composer.json`にあたるやつです。

ちなみに、`npm install`は`package.json`に基づいてdependencyがインストールされ、実際にインストールされたバージョンが`package.json`に書き込まれます。   
既に`package-lock.json`が存在する場合、基本的には`package-lock.json`に基づいてインストールされます。  
`package.json`内で指定されたバージョンと`package-lock.json`に矛盾があれば`package.json`が優先され、インストールされたバージョンが`package.json`に書き込まれます。

`package-lock.json`を優先させたい場合は、`npm install`の代わりに

```shell
npm ci
```
を実行します。

環境構築の際にこの辺りのパッケージのバージョンでトラブルが発生することが考えられるので基本的には`package-lock.json`が存在する場合はそちらを参照した方がよさそうです。

https://qiita.com/righteous/items/e5448cb2e7e11ab7d477

## Laravel Mixの使用
Laraavel Mixは先述した通り、webpack上の設定のためのパッケージなので、

`package.json`ファイル上のNPMスクリプトの1つをMixの実行で起動する。

公式サイトにはこんな感じで書かれてますが、要するにnpmコマンドで実行しろってことですね。
```shell
npm run dev
```
のような形で実行します。


以下のように
```shell
npm run watch
```
を実行することで関連ファイルが更新された際に自動で再コンパイルしてくれます。

## 実際の操作について

先述した通り、webpackの設定ファイルをより簡単に書くための仕組みがLaravel Mixですが、具体的にどう書いてどういう流れで処理されるのか、公式例等を用いて確認していきます。

公式
https://readouble.com/laravel/6.x/ja/mix.html

`webpack.mix.js`ファイルは全アセットコンパイルのエントリポイントです。
つまり、このファイルに設定を書くことでwebpackの設定が適用されるイメージです。

Mixを用いることで以下のファイルのコンパイルができます。

- CSS関係
  - Less
  - Sass
  - Stylus
  - PostCSS
  - 平文CSS

- JavaScript関係
  - ベンダの抽出
  - React
  - バニラJS
  - webpackカスタム設定

聞いたことのないワードもあるのでそれぞれ確認しながら、記述方法を確認します。

また、公式ドキュメントで以下の内容について触れています。

- ファイル/ディレクトリコピー
- バージョン付け/キャッシュ対策
- Browsersyncリロード
- 環境変数
- 通知


ここからは公式ドキュメントの内容をなぞりながら僕がわからなかったことについて調査したメモ書きになるので読み飛ばしてもらっても大丈夫です。(うまくまとめられなかった)

### Less

LessというCSSプリプロセッサがあります。
https://www.tohoho-web.com/ex/less.html
CSSをより簡単に書いてメンテナンスしやすくできるもの、といったイメージでしょうか。
Laravel Mixではこの`Less`をCSSへコンパイルすることができます。


```JavaScript
mix.less(元のlessファイル, 出力先)
```
のように記載することでコンパイルが可能です。

ちなみにLaravelではMixなどでコンパイルされたあとのデータを`public`配下に置きます。   
いろんな場所で部品を作ってそれを文字通りMixして出来上がったものを`public`に出力するイメージです。   
(初めてLarabelプロジェクトを見た時このフォルダの意味がわからなくて混乱してました。)


以下、公式ドキュメントの例を紹介します。
`resources/less/app.less`を`public/css/app.css`にコンパイルします。


単一ファイルの場合
```JavaScript
mix.less('resources/less/app.less', 'public/css');
```
複数ファイルの場合
```JavaScript
mix.less('resources/less/app.less', 'public/css')
    .less('resources/less/admin.less', 'public/css');
```
コンパイル後のファイル名を指定したい場合は第二引数に指定すればいいです。

### Sass

SassもLessと同じような形でコンパイルすることが可能です。

https://udemy.benesse.co.jp/design/web-design/sass.html

(Sassとscssなにが違うのかよくわかってなかったことがわかった。)

公式のコード例は以下

```php
mix.sass('resources/sass/app.scss', 'public/css');
```

### Stylus

Stylusも同様にコンパイルできるらしい。
Stylusが何かわからないので調べてみます。

https://qiita.com/morishitter/items/b9a2d78c79c3c07de776

>Node.js製のCSSプリプロセッサ。
SassとLessのいいとこ取りをしているらしい。

### PostCSS
CSS加工ツール

### 平文CSS
`styles`メソッドを使用することで平文のCSSを一つのファイルにまとめることもできます。
mix.styles([
    'public/css/vendor/normalize.css',
    'public/css/vendor/videojs.css'
], 'public/css/all.css');


### URL処理

>Webpackはスタイルシート中の`url()`呼び出しをリライトし、最適化します。
画像への相対パスを含むスタイルシートをコンパイルする際にURLを書き換えてくれる機能、といったイメージです。
例

```js
.example {
    background: url('../images/example.png');
}
```

デフォルトでは、パスを解決して`example.png`を見つけて、`public/images`フォルダにコピーします。

勝手に解釈させずに自分で指定したフォルダ構成を適用したい場合は下記のように記述することで`url()`リライトを停止できます。

```js
mix.sass('resources/assets/app/app.scss', 'public/css')
   .options({
      processCssUrls: false
   });
```

### ソースマップ
http://blog.sakurachiro.com/2017/08/scss_sourcemap/

使い方

```js
mix.js('resources/assets/js/app.js', 'public/js')
   .sourceMaps();
```

- ソースマップとは   
コンパイルする前のファイルを保持したもの。
コンパイルした後のファイルで不具合があった場合にコンパイル前のどの箇所でエラーが出たか特定しやすくなる。

### JavaScriptの操作

```js
mix.js('resources/assets/js/app.js', 'public/js');
```

このように書くことで以下の利点がある。(この辺りはLaravelのバージョンによって異なる？みたい)
- ES2015記法
- モジュール
- `.vue`ファイルのコンパイル
- 開発環境向けに圧縮

### バニラJS
スタイルシートのときと同様に、JavaScriptファイルをまとめることができる。
```js
mix.scripts([
    'public/js/admin.js',
    'public/js/dashboard.js'
], 'public/js/all.js');
```

### Browsersyncリロード
Browsersyncとは
自動的にファイルの変更を監視し、手動で再読み込みしなくても変更をブラウザに反映してくれる。
`mix.browserSync()`メソッドを呼び出し、有効にする。

公式例
```js
mix.browserSync('my-domain.test');

// もしくは

// https://browsersync.io/docs/options
mix.browserSync({
    proxy: 'my-domain.test'
});
```

`npm run watch`コマンドにより、webpackの開発サーバを起動します。
PHPファイルを変更すると、すぐにページが再読み込みされ、変更が反映されるのを目にする。


### 環境変数

`.env`ファイルの中でキーに`MIX_`をつけることで環境変数をMixへ注入できる。

```
MIX_SENTRY_DSN_PUBLIC=http://example.com
```

.envファイルで定義した内容にprocess.envオブジェクトを通してアクセスできるようになる。

```js
process.env.MIX_SENTRY_DSN_PUBLIC
```


## まとめ
今回は公式サイト+αといった形でうまくまとまらなかった部分がありますが、

- Laravel Mixを使うことでファイルの分割がしやすくなる(というより、分割した後の管理がしやすくなる)
- 特にスタイルシートやjsについては分割して管理できるとメリットが大きい

ことが掴めたかなと思います。
## 所感

普段使っている技術について調べていくと、他の技術との関連性や比較をする機会が出てくるので勉強になります。


詳細を追っていくと時間がいくらあっても足りないのでこのくらいで切り上げようと思いました。
実際使ってみてわかったことなどがあれば随時追記していきたいと思います。

cssのコンパイル一つとってもさまざまな方法があり、それぞれの長所があるので理解して使うことで長所を最大限活かせるなと感じました。

また、まだまだ知らない技術が多すぎてなかなか網羅的には調べきれていない部分があるのでもう少し局所的にまとめていく方がいいのかな？と悩んでいます。。。
あとはそろそろ手を動かして勉強していけるようになってきた、、、かな？と思ってます。

