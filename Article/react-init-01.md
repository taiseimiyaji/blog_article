---
title: React関連情報を漁る
tags: [react, memo]
createDate: 2023-05-07
updateDate: 2022-05-07
slug: react-init-01
draft: true
---

https://zenn.dev/ababup1192/articles/ebc5ff81f92a88

## React Server Components

https://zenn.dev/uhyo/articles/react-server-components-multi-stage

多段階計算

プログラムの評価を　他段階に分けて処理する機構
動的にコードを生成してそれを走らせる機構を備えた、計算が複数のステージからなる意味論を備えた体系(+それを安全に行うための型システム)

デフォルトでサーバー側のコンポーネント、stage0を使用する。クライアントはstage1。従来のreactはstage1のみが存在しており、そこにstage0を選択できるようになったという考え方


