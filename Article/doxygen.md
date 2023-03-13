---
title: できる限りドキュメントメンテナンスコストを下げたい [Doxygen] [phpDocumentor]
tags: [document]
createDate: 2022-05-13
updateDate: 2022-05-13
slug: doxygen
---

## はじめに(長めの前置き)
大前提として、「コードは必ず変更される」ことはエンジニアであれば理解されていると思います。   
経験上、コードを書いて1回切りの「作り捨て」にすることはまずありません。   
自分が変更するかもしれませんし、他人が変更することもあるでしょう。   
よく「三日後の自分は他人」とも言いますが、そう考えるとほとんどの場合、書いたコードを変更するのは「他人」ということになります。   
コードを書く際の判断や選択はこの前提を忘れないようにしましょう。   

ソフトウェアというのは複雑なものです。
完璧なソフトウェアというのは存在せず、障害が発生したり、ユーザーからの要望を反映しなければならないことがあります。

コードを理解するとき、その助けとなるのが各種設計書です。   
ただ、設計書は往々にして腐ります。   
コードはさまざまな要因で変更されますが、それに合わせて設計書を変更するのが手間だからです。

そこで、できるだけ楽をしてドキュメントを作成できれば、設計書のメンテナンスコストを下げることにつながると考え、ツールを使用してアプローチしてみたいと思います。

本記事では、PHPを使用したチーム開発環境を前提として、Doxygenとphpdocumentorを導入してみてその違いを触って理解してみます。

本記事は書いてある通りに実行すれば導入できるような親切な記事ではなく、実際に導入してみたいなと考えた時に必要な情報のみを記載します。

特にサンプルについてはあまり見当たらなかったのでよければ参考にしてください。
詳しい設定等は公式ドキュメントなどを参照してください。

## Doxygenのインストール

## 参考
https://qiita.com/hyt-sasaki/items/8f8312e277d1a4815ab6

## Doxygenとは
簡単にいうと、コードに記載されたコメントをもとに、設計書やクラス関係図などのドキュメントを生成してくれるツールです。

公式リンク http://www.doxygen.jp/

今回はDockerを用いたチーム開発を前提として環境構築してみます。

Docker Hubから`hytssk/doxygen`というイメージをpullしてきます。
```
docker pull hytssk/doxygen
```

### Doxyfileの作成
Doxygenは`Doxyfile`という設定ファイルを使用します。
下記コマンドを実行して生成します。
```
docker run --rm -v "${PWD}":/src hytssk/doxygen -g
```

内容の編集については公式ドキュメントを参照してください。(コメントを読むだけでもある程度わかります。)

参考サイト:

http://www.doxygen.jp/config.html

https://cercopes-z.com/Doxygen/list-config-dxy.html

### 動作確認用変更箇所
僕が動作確認した際に変更した箇所をメモしておきます。
もっと変えた方がいい箇所があるかもしれないです。
```
OUTPUT_LANGUAGE = Japanese
EXTRACT_ALL            = YES
WARN_LOGFILE           = "ログファイル名.txt"
INPUT                  = ソースコードのパス
RECURSIVE              = YES
SOURCE_BROWSER         = YES
GENERATE_LATEX         = NO
CALL_GRAPH             = YES
CALLER_GRAPH           = YES
```

### ドキュメント化
ソースコードにDoxygen用のコメントを記載した上で使用します。
以下コマンドを実行することでドキュメントが生成されます。

```
docker run --rm -v "${PWD}":/src hytssk/doxygen Doxyfile
```

## phpDocumentor

### phpDocumentorとは

Doxygenと同じくソースコードからドキュメントを生成するツールです。

公式Github

https://github.com/phpDocumentor/phpDocumentor

Dockerイメージ

https://hub.docker.com/r/phpdoc/phpdoc/

### 参考サイト
https://wand-ta.hatenablog.com/entry/2019/05/18/233631

http://blog.livedoor.jp/haruchaco/archives/1340012.html

### 実行

まずはDoxygenの時と同じくDocker hubからpullします。
```
docker pull phpdoc/phpdoc
```

ドキュメント生成
```
docker run --rm -v $(pwd):/data phpdoc/phpdoc -d [ソースコードのディレクトリ] -t [出力先のディレクトリ] 
```

以下のオプションを使用することでクラス図などの図が出力されます。

```
--setting=graphs.enabled=true
```

以下のオプションを使用すると出力形式を変更できます。
他にも種類があるようなので公式を参照してください。
```
--template="clean"
```



## 比較

### インストールの容易さ
phpdocumentorの方がインストールが簡単だという意見を見かけました。   
->まあ今回のようにDocker使うなら大して差はないです。

### 出力の違い
個人的にはphpDocumentorの方が新しいツールなのもあって出力結果のUIがいい感じだと思う。
DoxygenはPHPへの対応が少し弱いのでこの部分でphpDocumentorを選択するのはアリだと思います。
(ちなみにC言語とJavaでDoxygenを使用したことがあるがその時はいい感じだった。)

## サンプル
個人的に作ってるコードをドキュメント化してみました。   
簡単なValueObjectのみ作成しているが、Interface,abstractなどを使用してクラス図を確認できるようにしています。   
コメントにツール用の記述をすることでさらにドキュメントを充実させることができます。(というかそれが本来の使い方)   
ひとまず図を見たい人のためのサンプルだと思ってください。   
またサンプルコードが充実したら追記しようかな。

Doxygenのサンプル   
https://taiseimiyaji.github.io/doxygen_sample/

phpDocumentorのサンプル   
https://taiseimiyaji.github.io/phpDocumentor_sample/

## 所感

ドキュメントのメンテナンスはコストがかかるしめんどくさいものです。   
特に規模が小さい場合、わざわざドキュメントを作ることに後ろ向きになりがちだと思います。   
今回紹介したツールを使用すればさほど手間なくドキュメントを作成することができます。   

チーム開発の場合は個人の力量に差があることが多く、その差を埋める手助けになればと思います。   
また、上級者から初学者への説明の際の資料としても有用だと思いました。

個人的にはこういうめんどくさいなあと感じたことを自動化する瞬間が一番エンジニアであることを実感します。   

