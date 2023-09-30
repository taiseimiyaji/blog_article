---
title: LT会用のスライドを楽に作って楽に管理したい
tags: [LT]
createDate: 2022-07-01
updateDate: 2022-07-01
slug: marp-slides
---

## はじめに
弊社では現在、月一回のペースで社内LT会を行なっています。   
毎回観覧側として参加していましたが、入社して三ヶ月が経ち、自分でも発表してみようと思いました。   
そこで必要なのが発表スライドです。   
とは言ってもLT、つまりLightning Talksなので簡単なものでも大丈夫です。   
スライドを用意するためのツールをいろいろ考えていて、その中でも今回はgithubを用いて簡単にスライドを作成、管理ができそうなmarpを使ってみたいと思います。

## LTとは
本題に入る前にサッとLTについておさらいしておきます。   
LTとは、Lightning Talksの略です。   
稲妻のように短いプレゼンテーション。といったイメージです。   
基本的には何をどんなふうに発表してもOKで、時間だけが設定されていることが多いでしょうか。   
弊社のLT会もそんな感じのゆるいLT会となっています。

## Marpとは
公式ページ   
https://marp.app/

GitHub   
https://github.com/marp-team/marp

Markdownからプレゼン用のスライドを生成してくれるエコシステムです。

VSCodeを使用している場合は拡張機能が存在します。   
https://github.com/marp-team/marp-vscode

導入については以下のQiita記事が参考になるかもしれません。   
https://qiita.com/tomo_makes/items/aafae4021986553ae1d8

Marpを使用することのメリットは、Gitで管理しやすいことと共有しやすいことでしょうか。
他にスライドを作る方法としてGoogleスライド等も候補にあがりましたが、Gitで管理したいなと考えMarpを選んでいます。

## Marpの書き方
Markdownファイルを作成し、以下のような記述をします。
```
---
marp: true
---
```
たったこれだけでスライドが生成されるようになります。
VSCodeの拡張機能を使用した場合は以下のように簡単にプレビューを確認しながらスライドを作成できます。
僕が今回作成したスライドでは、この設定以外にも以下のような記述をしました。
```
---
marp: true
paginate: true
header: GASでランダムに司会を決定するSlackBotを作った話
footer: taisei miyaji
---

# GASでランダムに司会を決定するSlackBotを作った話

---
```
すると、スライドは以下のようになります。
![picture 1](/images/a6fe2ad9b0e874ec78a26812169390c6622bf47c07bb139bc3a8ce9e68958359.png)  


あとはMarkdownでスライドを作成していきます。


作成したスライドはこんな感じ。   
https://taiseimiyaji.github.io/slides/slides/SlackRandomBot/random.html

## 画像の貼り付けについて

個人的にスライドを作成する際に使用しているVSCodeの拡張機能があるので紹介しておきます。   
この拡張機能を使えばクリップボードにある画像を貼り付けた際に、WorkSpaceの下にimagesフォルダを作成して保存し、Markdownにリンクを書いてくれます。   
ちょっとしためんどくささを回避できる拡張機能です。   

https://marketplace.visualstudio.com/items?itemName=hancel.markdown-image


## GitHubでの運用
先ほどスライド例として添付した   
https://taiseimiyaji.github.io/slides/slides/SlackRandomBot/random.html

これはGitHub Pagesを使用してデプロイしています。
これをできるだけ自動化していきたいと思います。

## GitHub ActionsでMarp
まずはMarkdown->Marpでスライドを生成する工程を自動化します。   
自動化にはGitHub Actionsという仕組みが利用できそうです。   

### GitHub Actionsとは

https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions

公式を翻訳してみると以下のような記載があります。
```
GitHub Actionsは、ビルド、テスト、デプロイのパイプラインを自動化するためのCI/CDプラットフォームです。
リポジトリへのプルリクエストごとにビルドとテストを行うワークフローを作成したり、マージされたプルリクエストを本番環境にデプロイしたりすることができます。
```

まずはリポジトリにPushされると、自動でMarpを実行し、Markdownからスライドを生成してみます。

こちらの記事を参考に進めていきます。

https://zenn.dev/koharakazuya/articles/1abe9cb8d8f936

まずはリポジトリ直下に`.github/workflows`フォルダを生成します。
フォルダ内に任意の名前で`yml`ファイルを作成して以下の記述を追記します。
```
name: Publish GitHub Pages
on:
  push:
    branches:
      - main

jobs:
  publish:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v2

      - name: Convert Markdown into HTML and PDF
        uses: KoharaKazuya/marp-cli-action@v2
```

これでリポジトリへのPushをするたびにMarpを使用してPDFとHTMLが生成されます。   
ちなみに生成されるのは`gh-pages`というブランチなので注意してください。

## Github Pagesへのデプロイ

GitHub Pagesへのデプロイには先ほどの`yml`ファイルに以下の記述を追記します。
```
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./
```

これでGitHub Pagesにデプロイされます。   
フォルダ構成によりますが、indexを用意しておくと良いかもしれません。   
GitHub Pagesにはindex.htmlが最初に表示されるので、Marpでindex.htmlが生成されるようにindex.mdを作成します。

```
---
marp: true
---
# index
- [GASでランダムに司会を決定するSlackBotを作った話](slides/SlackRandomBot/random.html)

```

このようにindexにリンクを置いておくと管理もしやすく、共有する際もリンクの共有だけで簡単にスライドを共有できます。


## まとめ

- Markdownから簡単にスライドを作成できるMarp
- htmlファイルを簡単にデプロイできるGitHub Pages
- デプロイ時にパイプラインを自動化できるGitHub Actions
- 上記3つを組み合わせることでpushするだけで自動でスライドをデプロイできる。

## 所感

今回はスライド作成を楽にしたいと考え、いろんなツールを用いて実現しました。   
全くわからない状態からなんとなく使えるようにはなれたかなと思います。   
とはいえ、巨人の肩に乗っている感は否めないのでゆくゆくは自分でGitHub Actionsやツールを作成できればと思いました。   
簡単にスライドを作成することができるようになったのでどんどんLTも発表者として参加していきたいと思います。

