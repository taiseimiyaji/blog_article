---
title: astroに入門してみる
tags: [Astro]
createDate: 2023-02-04
updateDate: 2023-02-04
slug: astro-tutorial
---


## はじめに

最近フロントエンドへの自身の理解の甘さを感じるようになってきました。  
ここ最近TypeScriptなどフロントエンドに関係する記事をいくつか公開しましたが、そろそろ実際になにか作ってみようと感じるようになりました。  
そこで今回は気になっていた`astro`というフレームワークを触ってみようと思います。  
題材としては、自作ブログにしたいと思います。  
目的はブログ記事の管理を簡単にすることと、フロントエンドの学習、また、レンタルサーバーなどを用いてインフラ周りを含めた実践です。  

今回はフロントエンド関連のキーワードについて調べながらastroの公式ドキュメントを読み進めていきたいと思います。

## 3行まとめ

- astroはパフォーマンスを重視したMPAフレームワーク
- React, Vue, Svelteのような異なるフレームワークを使用できる
- MarkdownやMDXを使用したコンテンツ重視のWebサイト構築にはかなり向いている。


## astroとは

astroというフレームワークについて特徴をまとめておきます。

- 高速なWebサイトの構築のためのオールインワンWebフレームワーク
- サーバーファーストでサーバーサイドレンダリングを最大限活用する。
- アイランドアーキテクチャと呼ばれるアーキテクチャを採用している
- 高価なハイドレーションをデバイスから取り除く
- 様々なインテグレーションによってカスタマイズが可能
- ロードが遅くなる原因となる JavaScript をデフォルトではクライアントで起動しない
- React, Preact, Vue, Svelte, Solidなど様々なフレームワークをサポートしています。

### コアコンセプト

https://docs.astro.build/ja/concepts/mpa-vs-spa/

astroはMPAの考え方に沿ったフレームワークです。  
そのため、フロントエンドにおいてMPAとSPAの違いを再確認してみます。  

### MPA

マルチページアプリケーションの略。複数のHTMLページから構成されるWebサイトのこと。  
新しいページに移動すると、ブラウザはサーバーに新しいページのHTMLを要求します。  
従来のフレームワークの例としては`Ruby on Rails`や`Python Django`, `PHP Laravel`や`WordPress`なども静的サイトジェネレータです。  
ただ、astroがその他のMPAフレームワークと違う点はサーバー言語とランタイムにJavaScriptを使用している点です。  
結果として、Next.jsやその他のモダンWebフレームワークとよく似た感覚で、MPAサイトの利点であるパフォーマンス特性を備えた開発者体験が得られます。  

### SPA

シングルページアプリケーションの略。
ユーザーのブラウザに読み込まれ、ローカルでHTMLをレンダリングする単一のJavaScriptアプリケーションで構成されるウェブサイトのこと。  
ブラウザ上でJavaScriptアプリケーションとしてウェブサイトを実行し、ページ遷移したときに同じHTMLのページを再描画する機能が特徴的。
従来のフレームワークとしては`Nuxt.js`, `Next.js`, `SvelteKit`, `Gatsby`, `Create React App`などが挙げられます。  


MPAと比較した場合のメリット

- 最初のページロード以降はユーザー体験としてはMPAより良くなる。
- SPAはページレンダリングに関連する多くの制御を行うため、遷移時のアニメーションを提供する事ができる。MPAの場合は[Turbo](https://turbo.hotwired.dev/)のようなツールを使う必要がある。
- アプリケーションが複数のページに渡って状態とメモリを維持できるため、複雑な複数ページの状態管理を扱うWebサイトとして優れている。

MPAと比較した場合のデメリット

- ブラウザでHTMLをレンダリングするため、最初のページ読み込み時のパフォーマンスはMPAに劣る。
- シンプルさ、パフォーマンスではMPAノフが優れている。

### 結論

MPAとSPAでは「どちらが優れている」ということはなく、常にトレードオフの関係にある。

- 複雑な状態管理を必要とするページの場合はWebサイト全体を単一のJavaScriptアプリケーションのように扱えるSPAのほうが向いている  
- コンテンツに特化してシンプルさやパフォーマンスを重視するような場合はMPAのほうが向いていて、astroはMPA。  


## astroアイランド
https://docs.astro.build/ja/concepts/islands/

astroアイランドとはざっくりいうとHTMLの静的なページ上にあるインタラクティブなコンポーネントを指します。  
アイランドは常に独立したコンポーネントとして表示され、静的なコンテンツの海に浮かぶ島に比喩されます。  
astroでは、`React`や`Svelte`、`Vue`などのコンポーネントを使用して、ブラウザ上でアイランドをレンダリングする事ができます。  
アイランドの考え方から、同じページで様々なフレームワークを混在させることも可能です。  
astroはデフォルトではクライアントサイドでは一切JavaScriptを使用しません。  
`React`や`Svelte`、`Vue`などのコンポーネントを使用した場合には自動的に前もってHTMLとして生成し、JavaScriptを取り除いて表示します。  
インタラクティブにしたい部分のみを`clientディレクティブ`と呼ばれるものをつけて描画するよう指示します。  



### 参考
https://jasonformat.com/islands-architecture/


## astro2.0について

https://astro.build/blog/astro-2/
2023年の1月24日にastro2.0がリリースされました。  
特に大きなアップデートとして、MarkdownとMDXに型安全性が追加されました。  
WebでMarkdownを使用する場合に型安全に取り扱う事ができます。  

例えば、ブログ、ニュースレターなど多くのファイルが存在する場合にプロパティを設定しておくと一貫性を保つことができます。  
この一貫性のために、型安全性が追加されました。  

また、astro2.0にて追加されたハイブリッドレンダリングという機能で、静的コンテンツと動的コンテンツを組み合わせて使用することができます。  
これによって、既存の静的サイトにAPIを追加したり、ページのレンダリングパフォーマンスを改善する事ができます。  


## MarkdownとMDXについて

https://docs.astro.build/ja/guides/markdown-content/  
astroでは、インテグレーションをインストールすることで各種UIフレームワークを使用できるようになりますが、MDXファイルも対応しています。  

### MDXの機能について

ざっくり簡単なMDXの概念をまとめておきます。

公式: https://mdxjs.com/

参考: https://zenn.dev/spring_raining/articles/3eb62ff93df1eb

MDXはMarkdown + JSXを名前の由来としています。  
MDXを使用することで以下のメリットがあります。

- 簡単にJSXで作成した独自コンポーネントを使える
- 直接JavaScriptとして読み込める(JSXコンポーネントのように扱うことができる)

公式サイトから引用した例は下記の様になります。  
一見MarkdownにHTMLが埋め込まれたもののように見えます。

```md
export const Thing = () => <>World</>

# Hello <Thing />
```
これがJavaScriptに変換され、下記のようになります。

```jsx
/* @jsxRuntime automatic @jsxImportSource react */

export const Thing = () => <>World</>

export default function MDXContent() {
  return <h1>Hello <Thing /></h1>
}
```

変換後のコードを見てみると、exportが記載されており、コンポーネントとして出力されることがわかります。  

下記のようなmdxコンポーネントを作成し、
```md
# Hi!
```
下記のようにインポートできます。
```js
import React from 'react';
import ReactDom from 'react-dom';
import Example from './example.mdx';

ReactDom.render(<Example />, document.querySelector('#root'));
```

jsxとして出力されていますが、Reactに結合しているわけではなく、`Preact`, `Vue`, `Emotion`などでも使用できるようです。  
あくまでプログラミング言語ですが、jsxを理解できればあまり学習コストは高くないように感じました。

## astroにおけるルーティング
https://docs.astro.build/ja/guides/markdown-content/#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%83%99%E3%83%BC%E3%82%B9%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0

astroでは、先述したMarkdown及びMDXを使用することで簡単にルーティングを行うことができます。  
`src/pages`ディレクトリ内に`.md`ファイルもしくは`.mdx`ファイルを置くだけです。  
また、サブディレクトリ配下も自動的にページルートを構築します。  
また、front-matterを使用することで下書きページ扱いできたりもします。  

```md
---
title: タイトル
draft: true
---
```

markdownファイルの結果は`astro.glob()`関数にて取得できるので、ビルドに含めないようにフィルタリングすることで下書き機能が実現できます。  
`astro.glob()`については公式リファレンスを参照すると動きがわかりやすいです。  
https://docs.astro.build/en/reference/api-reference/

```js
const posts = await Astro.glob('../pages/post/*.md');
const nonDraftPosts = posts.filter((post) => !post.frontmatter.draft);
```

また、デフォルトでShikiとPrismといったシンタックスハイライトをサポートしているのもエンジニアからすると嬉しい機能です。  

## まとめ

- astroはパフォーマンスを重視したMPAフレームワーク
- React, Vue, Svelteのような異なるフレームワークを使用できる
- MarkdownやMDXを使用したコンテンツ重視のWebサイト構築にはかなり向いている。
- シンタックスハイライトやファイルベースルーティングなどmarkdownを中心にサイト構築する際に楽な要素が多い

## 所感

個人的には特にエンジニアにとって、技術ブログや個人サイトを構築する際に最有力になり得るフレームワークだと感じています。  
現在担当プロダクトでReactを導入しているプロダクトはないのですが、これを機にastroでReactを使ってみたいと思っています。  
現在主流のNext.jsとはすこし方針が違いますが、個人のブログではこちらのほうが向いてそうだなと感じています。(いい意味で少し変態的な部分に惹かれています)  

また、いままで個人サイトの構築経験がなかったため、ドメインの取得やサーバーへのデプロイなど含めてチャレンジしてみようと思ってます。  
最近は下記のリポジトリでastroのチュートリアルを試してみたり、ファイルベースルーティングなどを試しながら遊んでいます。  
デプロイ方法としてはRoute53でドメインを購入して、astro公式ドキュメント通りNetlifyにデプロイしています。  
astroを布教して社内でもエンジニアの技術発信がもっと活発になればいいなと思ってます。  

リポジトリ
https://github.com/taiseimiyaji/astro-blog

デプロイ先
https://www.lyricrime.com/

