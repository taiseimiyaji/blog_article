---
title: tRPCへの入門
tags: [tRPC]
createDate: 2024-09-23
updateDate: 2024-09-23
slug: trpc-init
draft: false
---

## はじめに

今回はtRPCという技術について調査してみます。  
なにもわからない状態からのやってみた系記事になります。  
概念の理解や個人的な感想を中心に書いていきます。  
実際に導入する際には公式サイトを参照することをおすすめします。  

## tRPCとは

そもそもRPCとは?については[こちらのQiita記事](https://qiita.com/il-m-yamagishi/items/8709de06be33e7051fd2)に詳しく書かれています。

tRPC公式サイトにも[コンセプトページ](https://trpc.io/docs/concepts)が用意されています。

tRPCを利用するメリットとしては、TypeScriptによる型推論を活用して、フルスタックアプリケーションを構築する場合において、型安全なAPIクライアントとサーバーを提供することができる点が挙げられます。[公式サイト](https://trpc.io/)には以下のような特徴が記載されています。日本語訳なので不自然な表現があるかもしれませんが、ご了承ください。

1. 自動的安全性: サーバー側で変更を加えた場合、ファイルを保存する前にクライアント上でエラーを警告します。
2. SnappyDX: tRPCにはビルドやコンパイルのステップがなく、コード生成、ランタイムの膨張がありません。(Snappyは快活な、活発な、きびきびした、威勢の良い、てきぱきしたという意味があります。)
3. フレームワークに依存しない: 全てのJavaScriptフレームワークおよびランタイムと互換性があります。既存のプロジェクトに簡単に追加できます
4. 自動補完: tRPCを使用すると、APIのサーバーコードにSDKを使用するのと同じになり、エンドポイントに自信が持てるようになります。
5. ライトバンドルサイズ: tRPCには依存関係がなく、クライアント側のフットプリントが小さいため軽量です。(フットプリントとは、稼働時に必要とする資源の大きさという意味があります。)
6. 電池付属: React, Next.js, Express, Fastify, AWS Lambda、Solid、Svelteなどのアダプターを提供しています。

## tRPCを導入する手順

### 1. tRPCのセットアップ

パッケージのインストール

```shell
npm install @trpc/server@next @trpc/client@next
```

今回はNext.jsにtRPCを導入してみます。まずはsrc配下にserverディレクトリを作成し、trpc.tsを作成します。
ここではバックエンドを初期化します。
いい慣例として、再利用可能なヘルパー関数としてエクスポートすることが推奨されています。

`src/server/tprc.ts`

```typescript
import { initTRPC } from '@trpc/server';

const t = initTRPC.create();

export const router = t.router;
export const publicProcedure = t.procedure;
```

次にルーターのセットアップをします。

公式サイトでは`server/index.ts`に記述されていますが、今回は`src/server/routers`フォルダを用意し、エンドポイントが増えた場合に対応しやすいように記述します。

`src/server/routers/_app.ts`

```typescript
import { router } from '../trpc';
import { exampleRouter } from "./example";

const appRouter = router({
  // ここにルーターを追加
  example: exampleRouter
});

export type AppRouter = typeof appRouter;
```

### 2. エンドポイントの作成

それぞれのエンドポイントでは、下記のようにプロシージャを作成します。

`src/server/routers/example.ts`

```typescript
import { publicProcedure, router} from "../trpc";
import { z } from "zod";

// エンドポイントの定義

export const exampleRouter = router({
  hello: publicProcedure
    .input(z.object({
      text: z.string().nullish()
    }).nullish())
      .query(({ input })=> {
        return {
          greeting: `Hello, ${input?.text ?? "world"}!`
        }
      }
    )
});
```

これはごく簡単な例ですが、この内容がエンドポイントの処理内容になるので、ドメイン駆動設計やクリーンアーキテクチャなどを取り入れる場合はさらにファイルを分割することになります。

このプロシージャーをAction層、Controller層として扱い、UseCaseやRepository、Service、ドメインオブジェクトなどに分割してそれぞれテストを書いていくことができます。

この例ではシンプルに書いてますが、実際にはprisma等を利用してDBアクセスしたり、外部APIを呼び出したりすることができます。

### 3. クライアントのセットアップ

クライアント側のコードに移り、バックエンドと同じ型を利用し型安全性の力を活用しつつ、バックエンドの呼び出しを行います。

tRPCのセットアップとして、公式サイトのクイックスタート同様にhttpBatchLinkを利用します。解説は後に回します。

`src/client/trpc.ts`

```typescript
import { createTRPCClient } from '@trpc/client';
import type { AppRouter } from 'src/server/routers/_app';
import { httpBatchLink} from "@trpc/client";

export const trpc = createTRPCClient<AppRouter>({
      links: [
        httpBatchLink({
          url: '/api/trpc'
        })
      ],
    }
);
```

### 4. クライアントからエンドポイントを呼び出す

クライアントの任意のファイルでエンドポイントを呼び出します。

```typescript
    const example = await trpc.example.hello.query({
      text: "世界"
    });
```

### httpBatchLinkについて

httpBatchLinkは、複数のリクエストを一度に送信するためのリンクです。

例えば、次のようなリクエストを送信することができます。

```typescript
const somePosts = await Promise.all([
    trpc.post.byId.query({ id: 1 }),
    trpc.post.byId.query({ id: 2 }),
    trpc.post.byId.query({ id: 3 }),
])
```

このコードは1つのHTTPリクエストになります。

詳細は[公式サイト](https://trpc.io/docs/client/links/httpBatchLink)を参照してください。

### 用語について

改めて公式サイトに記載されているコンセプトと用語を確認しておきましょう。

[RPCとは](https://trpc.io/docs/concepts#what-is-rpc-what-mindset-should-i-adopt)

RPCはRemote Procedure Callの略です。あるコンピューター上の関数を別のコンピューターから呼び出すことができる方法のことを指します。(厳密にはより広義の意味合いもありますが、現代フロントエンドにおいてはこの意味合いが一般的です。)  

tPRCは、TypeScriptのモノレポように設計されたRPCの実装の1つです。

平たく言えば、HTTP通信のラッパーであり、アプリケーションコードを書く際に実装の詳細について意識せずに、関数を呼び出すだけでtRPCが全てを処理します。  

公式サイトより、以下の用語がエコシステムで頻繁に使用される用語です。

| 用語 | 意味 |
| --- | --- |
| Procedure | APIエンドポイント - `query`、`mutation`、`subscription`のいずれか。 |
| Query | データを取得するための手続き。procedure |
| Mutation | データを変更するための手続き。creates, updates, or deletes を行う手続き。procedure |
| Subscription | 持続的な接続を作成し、変更を受け付けるための手続き。procedure |
| Router | 共有している名前空間の下にあるprocedure(または他のルーター)の集まり。 |
| Context | 全てのprocedureがアクセスできるもの。セッション状態やデータベース接続などに使用される。 |
| Middleware | procedureの前後に実行できる関数。コンテキストを変更することができる。 |
| Validation | procedureの入力データを検証するためのしくみ。 |

## まとめ

- tRPCはTypeScriptの型推論を活用して、フルスタックアプリケーションを構築する場合において、型安全なAPIクライアントとサーバーを提供することができる。
- サーバーサイドのエンドポイントは、プロシージャを作成し、アクション層、コントローラー層として扱い、UseCaseやRepository、Service、ドメインオブジェクトなどに分割してそれぞれテストを書いていくこともできる。
- クライアントサイドからは関数を呼び出すだけでtRPCの処理を呼び出すことができる。
- Next.jsと組み合わせて使う場合は、API Routeを利用することで、サーバーサイドのエンドポイントを作成することができる。
