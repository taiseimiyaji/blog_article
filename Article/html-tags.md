---
title: HTMLタグとアクセシビリティ、コンポーネント化について考える
tags: [html]
createDate: 2024-05-30
updateDate: 2024-05-30
slug: html-tags
draft: false
---

## はじめに

最近、個人的なプロジェクトのコードをNext.jsで書きながらキャッチアップをしています。  
普段の業務では、すでにコンポーネント化されたUIを組み合わせて利用することが多く、基本的なコンポーネントを1から作ることはないのですが、個人で開発する場合には、コンポーネント化したくなる場面があり、その方針や基準に悩む機会が増えました。  
今回は、基本的なHTMLタグを復習しつつ、コンポーネント化の方針について考えてみたいと思います。  

## ユーザーのアクセシビリティに影響するHTMLタグ

基本的に、HTMLタグはそのタグ自体が持つ意味を持つように設計されています。
コンポーネント化したいケースの大半は、コンポーネントの挙動、インタラクションと呼ばれる部分に関わることが多いですが、HTMLタグには、そのタグ自体が持つ意味があり、それを無視してしまうと、ユーザーのアクセシビリティに影響を与えることがあります。

まずはインタラクションを実現することのできる要素について、復習してみます。

## button,form,label

### button

`<button>`要素は、ユーザーがクリックできるボタンを作成するための要素です。

例:

```html
<button type="button">Click Me</button>
<button type="submit">Submit</button>
<button type="reset">Reset</button>
```

- `type="button"`: ボタンをクリックしたときに何もしない。この要素のイベントを待ち受けし、イベントが発生すると起動されるスクリプトを設定すること亜できます。
- `type="submit"`: ボタンをクリックしたときに、フォームのデータを送信します。
- `type="reset"`: ボタンをクリックしたときに、フォームのデータをリセットします。

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/button](https://developer.mozilla.org/ja/docs/Web/HTML/Element/button)

### form

`<form>`要素は、ユーザーからの入力を受け付け、サーバーに送信するためのコンテナです。

例:

```html
<form action="/submit" method="post">
  <label for="name">Name:</label>
  <input type="text" id="name" name="name">
  <button type="submit">Submit</button>
</form>
```

このように、buttonやinputといった要素を`<form>`要素内に配置することで、ユーザーからの入力を受け付け、サーバーに送信することができます。

- `action` 属性：フォームデータを送信するURLを指定します。この値は `<button>`、`<input type="submit">`、`<input type="image">` の formaction 属性によって上書きすることが可能です。この属性は method="dialog" が設定されている場合は無視されます。
- `method` 属性：データ送信の方法を指定（GET または POST）できます。また、dialog要素の中にformがある場合はdialogを指定することでdialogを閉じてsubmitイベントを発火させることができます。

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/form](https://developer.mozilla.org/ja/docs/Web/HTML/Element/form)

### label

`<label>` 要素は、フォーム入力要素のラベルを定義します。ラベルをクリックすると対応する入力フィールドにフォーカスが移ります。

例:

```html
<label for="email">Email:</label>
<input type="email" id="email" name="email">
```

関連づけておくと、スクリーンリーダーなどの支援技術を使っているユーザーにとって、フォームの入力フィールドが何を意味しているのかを理解しやすくなります。

ちなみに、`<label>`要素の中に`<input>`要素を配置することもでき、この場合はfor属性やid属性を使わずに関連づけることができます。

```html
<label>
  Email:
  <input type="email" name="email">
</label>
```

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/label](https://developer.mozilla.org/ja/docs/Web/HTML/Element/label)

## input

`<input>` 要素は、多様な種類のデータをユーザーから入力させるための要素です。

```html
<label for="username">Username:</label>
<input type="text" id="username" name="username">

<label for="password">Password:</label>
<input type="password" id="password" name="password">

<label for="email">Email:</label>
<input type="email" id="email" name="email">

<label for="birthday">Birthday:</label>
<input type="date" id="birthday" name="birthday">

<button type="submit">Submit</button>
```

type属性には、text, password, email, date, number, tel, urlなどさまざまな値を指定できますが、ここでは割愛します。

関連するトピックとして最近目にしたのは[今どきの入力フォームはこう書く！HTMLコーダーがおさえるべきinputタグの書き方まとめ](https://ics.media/entry/11221/)という記事です。

また、input要素でワンタイムパスワードを実装する場合は、`<input type="password">`ではなく、`<input type="text" autocomplete="one-time-code">`を使用することで、自動入力で煩わしさを感じることがなくなりそうです。([参考](https://x.com/honey321998/status/1795400591537312074))

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/input](https://developer.mozilla.org/ja/docs/Web/HTML/Element/input)

## textarea

`<textarea>`要素は、複数行のテキスト入力フィールドを提供します。

```html
<label for="message">Message:</label>
<textarea id="message" name="message" rows="4" cols="50"></textarea>
```

- rows 属性：行数を指定
- cols 属性：列数を指定

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/textarea](https://developer.mozilla.org/ja/docs/Web/HTML/Element/textarea)

## details,summary

`<details>` 要素は、ユーザーがクリックすると折りたたまれた内容を表示するための要素です。

```html
<details>
  <summary>More Info</summary>
  <p>This is additional information that can be revealed by clicking on "More Info".</p>
</details>
```

`<summary>`要素は、`<details>`要素の折りたたみヘッダーを定義します。  
`<summary>`要素がない場合は通常は「詳細」という文字列を表示します。  

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/details](https://developer.mozilla.org/ja/docs/Web/HTML/Element/details)

## a,nav

### a

`<a>`要素は、ハイパーリンクを作成するための要素です。

```html
<a href="https://www.example.com">Visit Example</a>
```

- href 属性：リンク先のURLを指定します。

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/a](https://developer.mozilla.org/ja/docs/Web/HTML/Element/a)

### nav

`<nav>` 要素は、ページ内の主要なナビゲーションリンクのセクションを定義するために使用されます。`<nav>` 要素自体はリンクではなく、ナビゲーションリンクをグループ化するためのコンテナとして機能します。

```html
<nav>
  <a href="/home">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>
```

MDNには、下記の記載があり、アクセシビリティに配慮したナビゲーションの作成に役立つかもしれません。  

>スクリーンリーダーのような障碍者向けのユーザーエージェントは、この要素を使用してナビゲーション用のコンテンツを初期読み上げから省略するかを判断するために使用することがあります。

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/nav](https://developer.mozilla.org/ja/docs/Web/HTML/Element/nav)

## select,option

`<select>`要素は、ドロップダウンリストを作成します。  
`<option>` 要素は、`<select>` 要素内で選択肢を定義します。  

例:

```html
<label for="choice">Choose an option:</label>
<select id="choice" name="choice">
  <option value="option1">Option 1</option>
  <option value="option2">Option 2</option>
  <option value="option3">Option 3</option>
</select>
```

[https://developer.mozilla.org/ja/docs/Web/HTML/Element/select](https://developer.mozilla.org/ja/docs/Web/HTML/Element/select)

## コンポーネント化の方針

フロントエンドをがっつり触ったことがあまりないですが、基本的な方針としては、上記のようなHTMLタグを適切に利用したうえで、デザイン含めて再利用性を高めるためにコンポーネント化を行うことが望ましいのかなと感じました。  
単純なインタラクションのみの場合は、HTMLタグを適切に使い、コンポーネント化を行わない方が良い場合もあるかもしれません。  
個人的にコンポーネント化するか考えるときは、以下のような観点で判断しています。

- 再利用するかどうか
- 複雑かどうか
- 状態を保つかどうか
- デザインの変更があるかどうか

## コンポーネントの分割

コンポーネント化と同時に、コンポーネントの分割についても考える必要があります。  
また、利用するライブラリによっても方針が変わるかもしれません。例えば、Reactも場合はJSX内で参考演算子を使ってViewを出し分けたい場合にロジックとViewが混ざってしまい、見づらくなるのでコンポーネント分割を行うことが多いです。  
Vueの場合はv-if等の仕組みがあるので、この理由で分割することは少ない気がします。  
また、バックエンドのクラス分割と同様に、責務やライフサイクル基準で分割することもあります。  

## 所感

近年のフロントエンドでは、技術の進化が著しく、新しいライブラリやフレームワークが次々と登場しています。  
それらのキャッチアップに加えて、アクセシビリティへの意識が高まっている印象があります。  
例えば、下記のようなブログを各社が公開していることもあります。  

- [Qiitaアクセシビリティ史 ~ Qiitaの歩んできた道 ~](https://qiita.com/degudegu2510/items/4313f361d46e31a53420)  
- [だれもが使える設定画面に─kintoneのフロントエンド刷新とアクセシビリティ](https://note.com/cybozu_design/n/na9e33dd9a913)

個人的にもフロントエンドの改善に課題感を持っているため、この辺りは興味深いです。  
生成AIの進化等によって、SEOハック等のために本来の挙動以外のHTMLタグを使うことが減るのではないかと思っています。  
アクセシビリティに配慮したコードを書く第一歩は、HTMLの仕様を詳細まで理解し、意図通りのタグを使うことだと感じました。  

フロントエンドの難しいところは、JS、HTML、CSSをそれぞれ実装することですが、これらがお互いの領域と重なる部分があり、HTMLで実現できることと同様の動作をJSで実装してしまえたり、CSSで実現できることをJSで実装してしまったりすることがあります。  

また、コンポーネント分割やライブラリのキャッチアップなどのコード面とデザイン面の両方を考える必要があるため、フロントエンドは複雑な部分が多いと感じます。  

アクセシビリティやフロントエンドはまだまだ勉強し始めたばかりなので、これからも学び続けていきたいです。
