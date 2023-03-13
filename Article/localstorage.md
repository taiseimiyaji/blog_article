---
title: LocalStorageとは
tags: []
createDate: 2022-07-28
updateDate: 2022-07-28
slug: localstorage
---

## LocalStorageとは
参考:
https://developer.mozilla.org/ja/docs/Web/API/Web_Storage_API

WebStorageAPIの一種
ブラウザがキーと値のペアを安全に保存できる仕組み。

2種類の仕組みがある。

- sessionStorage
- localStorage

### sessionStorage

- セッションデータを保存する。タブが閉じられるとデータは消去される
- データがサーバーに転送されることはない
- ストレージは最大5MBとcookieと比べると大きい。(cookieは4KB)

### localStorage

- 有効期限なし、クリアするにはJavaScriptを実行する必要がある。


これらはHTML5から導入された技術です。
それまではcookieやセッションといった方法しかありませんでした。

文字列のみを保存できます。

公式を見ていただくのが確実ですが、基本的な使い方について簡単にまとめておきます。

## データの保存

```JavaScript
localStorage.setItem('key', 'value');
```

文字列として保存するので、
```JavaScript
localStorage.setItem('number', 1);
```
のように保存しても文字列として保存されます。

## データの取得
```JavaScript
const value = localStorage.getItem('key');
```
キーを指定して`getItem`メソッドで取得します。

## データの削除

```JavaScript
localStorage.removeItem('key');
```
キーを指定して特定のvalueを削除するには`removeItem`メソッドを使用します。

## 全てのデータを削除する

```JavaScript
localStorage.clear();
```
localStorageのデータを全て削除します。
全部削除されるので基本的に使用するのは避けた方がいいかなと思います。
削除される範囲について調べてみます。

## localStorageの範囲

localStorageは同じドメイン内でのみ有効。
サブドメインやポート番号が違う場合はlocalStorageも別で管理される。

`hoge.fuga.com`と`hogehoge.fuga.com`ではサブドメインが違うのでlocalStorageは別扱いとなります。
また、`hoge.fuga.com`と`hoge.fuga.com:8000`ではポート番号が違うので別扱いです。

## 配列やオブジェクトを保存する

```JavaScript
const object = {
    key1: 'value1',
    key2: 'value2',
    key3: 'value3',
}
localStorage.setItem('key', JSON.stringify(data));
```

## 配列やオブジェクトを取得する

```JavaScript
const object = JSON.parse(localStorage.getItem('key'))
```

## localStorageの使用用途
サーバーサイドに保存するほどではないデータ、例えば検索時の絞り込みの設定やブログ等にコメントする際の名前等でしょうか。
ただ、初めに説明した通り半永久的に有効なデータであるため、管理するのが大変になります。
規模が大きくなるに従ってグローバル関数と同様にどのタイミングでアクセスされているか管理するのが大変になります。
クライアントサイドに保存されるためセキュリティ上のリスクに気をつけて扱う必要があります。

## セキュリティについて
localStorageについてまとめようとしていたのですが思っていたより簡単に扱えて、記事に書くことがなくなってしまったのでセキュリティについても調べてみました。
参考:
https://techracho.bpsinc.jp/hachi8833/2019_10_09/80851

参考記事によると

- 重要な情報を一切含まない
- ハイパフォーマンスを要求されない
- 5MB以内に収まるデータ量
- 文字列のみのデータ

の条件を満たす、公開されても問題ない情報の場合でのみ利用しても良いと書かれています。


OWASP（Open Web Application Security Project)のlocalStorageについての解説にも書かれていました。

https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/HTML5_Security_Cheat_Sheet.md#local-storage

- 永続ストレージが必要ない場合は基本的にsessionStorageを使用する。
- XSSでこれらのオブジェクト全てのデータを盗むことができるため機密情報はローカルストレージには保存しない。
- XSSでこれらのオブジェクトに悪意のあるデータをロードすることもできるため、これらのオブジェクトを信頼しない
- データは常にJavaScriptでアクセスできるため、セッション識別子をローカルストレージに保存しない。`httpOnly`Cookieを利用するとリスクを軽減できる。

基本的に公開されても問題のないデータのみを扱うべきで、セッションの識別子等トークンについてもlocalStorageを利用することは避けた方がよさそうです。
この辺りの議論はいろんな記事があって混乱するのですがlocalStorageの仕組みを考えると機密情報は扱わない方がいいと思いました。

## まとめ
Cookie

- サーバーからの指示でブラウザ上に保存される
- アクセスのたびにサーバーに送信される

localStorage

- JavaScript操作によってブラウザに保存される
- シンプルなキーバリューストレージでサーバーには自動送信されない
- アクセスできる範囲は同一オリジンのみで消去しない限り半永久的に残り続ける

## 所感
今回はlocalStorage自体がシンプルな仕組みなのでかなりライトな記事となりました。
問題はシンプルが故の管理のしづらさとセキュリティ面でした。
セキュリティについてまだまだ知識不足な部分があるので引き続き勉強したいと思います。
以下のスライドがめちゃくちゃ良さそうで中身みてみましたがまだ全然理解できていないので一つずつ勉強していきたいです。

https://www.slideshare.net/ockeghem/phpconf2021spasecurity