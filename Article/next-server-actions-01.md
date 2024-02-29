---
title: Next.jsのServerActionsを試してみる
tags: [Next.js]
createDate: 2024-02-29
updateDate: 2024-02-29
slug: next-server-actions-01
---

## はじめに

ここ最近あまり手を動かして新しいものを作成することができていなかったので、今回はNext.jsのServerActionsを試して見たいと思います。

まずは簡単なTodoリストの作成から始めてみます。

## 開発方針

個人開発ですが、先にデプロイの方法を考えてから開発を始めてみます。

今回選択したデプロイ先はVercelのHobbyプラン + PlanetScaleです。

どちらも無料枠があるので、個人開発を手軽に始めるには良い選択肢かなと思いました。

## 利用するFW、ライブラリ

今回はServerActionsとNext.jsを使ってみたいので、Next.jsの14.0.4、app routerを利用します。

その他インストールするライブラリは以下の通りです。

```json
"@prisma/client": "^5.10.2",
"next": "14.0.4",
"next-auth": "^4.24.5",
"prisma": "^5.10.2",
"react": "^18",
"react-dom": "^18"
"autoprefixer": "^10.0.1",
"biome": "^0.3.3",
"postcss": "^8",
"tailwindcss": "^3.3.0",
"typescript": "^5"
```

## デプロイ先の設定

今回はとりあえずデプロイして動作すればOKを目標にするので、Vercel上で先に行っておく設定としては環境変数の設定くらいです。

ローカルでは`.env.local`に設定しておき、production環境用の環境変数はVercelのダッシュボードから設定します。

next-authを利用したGoogleのOauth認証を作成したので、以下の環境変数も併せて設定しておきます。

```env
DATABASE_URL= // planetScaleのDB URL
GOOGLE_CLIENT_ID= // Google OauthのClient ID
GOOGLE_CLIENT_SECRET= // Google OauthのClient Secret
NEXTAUTH_URL=
NEXTAUTH_SECRET=
```

## データベースの設定

今回はORMとしてPrismaを利用するので、まずはPrismaの設定を行います。  

この辺りは公式ドキュメントを参照しながら進めていきます。  
公式ドキュメントではPlanetScaleに関する情報も記載されています。

[公式ドキュメント](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-planetscale)

割とセットアップする機会が多いのでChatGPTの回答を参考にして進めても詰まることなく進めることができました。

インストール

```bash
npm install prisma @prisma/client
```

初期化

```bash
npx prisma init
```

この時点でprismaディレククトリが作成され、`schema.prisma`が作成されます。

また、.envファイルを作成していない場合は作成されるはずです。

先程のあげた内容の環境変数を設定し、DATABASE_URLにはPlanetScaleのURLを設定します。

以下のような形式でPlanetScaleからURLが提供されるので、それを設定します。

```env
DATABASE_URL="mysql://<username>:<password>@<host>/<database>?sslaccept=strict"
```

## データベースのマイグレーション

PlanetScaleはgitブランチのような仕組みでDBスキーマを管理するため、従来のマイグレーションとは少し異なる形でマイグレーションを行います。

まずは`schema.prisma`を編集して、テーブルを作成します。

Prismaの`db push`を利用してローカルのスキーマ変更をPlanetScaleのブランチにプッシュします。

そのうえでPlanetScale上のダッシュボードを変更内容を確認し、問題がなければメインブランチのDBスキーマとしてマージします。

この辺りはPrismaの従来の方法ではないので少し注意しながら作業する必要があります。

## ServerActionsの作成

本題のServerActionsの作成です。

### ServerActionsの概要

ざっくりとした概要をChatGPTに聞いてみました。

```markdown

Next.jsのServerActionsは、サーバーサイドで非同期関数を実行するための機能です。これらは、サーバーおよびクライアントコンポーネントでフォームの送信やデータの変更を処理するために使用できます。

### 概要

- **定義と使用法**: ServerActionsは`async`関数として定義され、`"use server"`ディレクティブを使用してマークされます。これらのアクションは、サーバーコンポーネントおよびクライアントコンポーネントで使用でき、フォームの送信やデータの変更を処理するために使用されます。

- **動作**: ServerActionsは`<form>`要素の`action`属性を使用して呼び出され、サーバーコンポーネントではプログレッシブエンハンスメントがサポートされています。クライアントコンポーネントでは、JavaScriptがロードされるまでフォームの送信がキューに入れられ、ハイドレーション後にブラウザのリフレッシュが発生しないようになっています。

Next.jsのServerActionsは、サーバーサイドで非同期関数を実行するための機能です。これらは、サーバーおよびクライアントコンポーネントでフォームの送信やデータの変更を処理するために使用できます。

- **クライアントコンポーネントでの使用**: クライアントコンポーネントでは、モジュールレベルの`"use server"`ディレクティブを使用してインポートされたアクションのみを使用できます。ServerActionをクライアントコンポーネントにプロパティとして渡すこともできます。


### ServerActionsの利点

- **サーバーサイドのデータ取得**: ServerActionsを使用すると、データ取得をサーバーに移動し、データソースに近づけることができます。これにより、レンダリングに必要なデータの取得時間が短縮され、クライアントが行うリクエストの数が減り、パフォーマンスが向上します。

- **セキュリティ**: ServerActionsを使用すると、トークンやAPIキーなどの機密データやロジックをサーバーに保持できます。これにより、クライアントにこれらの情報が露出するリスクを回避できます。

- **キャッシュ**: サーバー上でレンダリングすることで、結果をキャッシュして後続のリクエストやユーザー間で再利用できます。これにより、パフォーマンスが向上し、各リクエストでのレンダリングおよびデータ取得の量が減少し、コストが削減されます。

```

ただ、今回の実装にあたってChatGPTの回答はあまり精度が高くなく、かえって混乱することが多かったので、Next.jsの公式ドキュメントを参照しながら進めて行った方が良さそうです。  

技術要素でいうとPrismaに関する情報は比較的精度が高かったんですが、他の技術要素に関してはChatGPTの利用を避けた方が良さそうな印象を受けました。

### ServerActionsの実装

app routerの場合はルーティング設定がpages routerの場合とは異なる点に留意しつつ、ServerActionsを実装していきます。

pages routerを触ったことがある場合は下記のようなZennの記事を見るとイメージ掴みやすいかもしれません。

[ざっくりApp Router入門【Next.js】](https://zenn.dev/yamadadayo123/articles/6cb4f586de0183)

pages routerにも自信がなかったりする場合は公式ドキュメントを端から読んでいくほうが近道かもしれません。

app/todos/action.ts

```typescript
"use server";
import { PrismaClient } from "@prisma/client";
import type { NextRequest } from "next/server";

const prisma = new PrismaClient();

export async function getTodos() {
	"use server";
	const todos = await prisma.todo.findMany({
		orderBy: { createdAt: "desc" },
	});
    return todos;
}

export async function createTodo(formData: FormData) {
	"use server";
	const rawFormData = {
		title: formData.get("title") as string,
		description: formData.get("description"),
		completed: formData.get("completed") === "true",
	};
	const todo = await prisma.todo.create({
		data: { title: rawFormData.title, completed: false },
	});
}
```

このような形で、"use server"を文頭につけます。  
また、引数にはformDataを受け取るとform内のinput要素の値を取得することができます。

今回は極力シンプルな形で実装しましたが、実際にはバリデーションやエラーハンドリングなども実装する必要があります。

参考: [【Next.js】Server Actionsを現場で使うテクニック](https://zenn.dev/rio_dev/articles/eb69fae0557f20)

コンポーネント側の実装としてはform要素のactionに上記の関数を指定することで、ServerActionsを利用することができます。

```tsx
	const handleCreate = async () => {
		const formData = new FormData();
		formData.append("title", newTodo);
		try {
			await createTodo(formData);
		} catch (e) {
			console.error(e);
		}
		// 作成完了後にリロードする
		window.location.reload();
	};
```

今回の場合はJSの関数を利用したかったので、クライアントコンポーネントから上記の呼び出しを行っています。

ただ、ServerActionsのメリットを活かせないような気もしているのでもう少しドキュメントを読みつつ試行錯誤してみたいと思います。  

戸惑う点としては、フロントエンドの中でもクライアントサイドとサーバーサイドのコードを混在させることになるので、意識の切り替えが自分の中でうまくできていない感触がありました。

ServerActions、RSCあたりはまだなにもわからないので、思想を理解できるように手を動かしていきたいと思います。

参考[一言で理解するReact Server Components](https://zenn.dev/uhyo/articles/react-server-components-multi-stage)

## まとめ

今回はNext.jsのServerActionsを試しつつ、新しく個人開発を始めるにあたっての最小限のデプロイまで行いました。  

フロントエンドはまだまだ知見が浅くベストプラクティスがわからない分野なので、悩みながらも楽しくすすめることができました。

個人的には先にデプロイをしやすい環境を用意しておくとモチベーションを保ったまま開発を進めることができているかなと思います。

今後も引き続きフロントエンドもキャッチアップをしつつ、個人開発を楽しみたいと思います。
