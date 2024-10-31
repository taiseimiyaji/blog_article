---
title: 2024年にReactを学ぶ人のための資料
tags: [React]
createDate: 2024-10-31
updateDate: 2024-10-31
slug: 2024-start-react
---

## はじめに

2024年現在、WebフロントエンドはReactが主流となっています。  
LaravelやRuby on Railsのようなフルスタックフレームワークの流行を過ぎ、バックエンドとフロントエンドを分離することが一般的になりました。  
今回は、すでにjQueryやLaravel、Ruby on Railsなどを触ったことがある人を対象に、Reactを学ぶための資料をまとめてみました。  
個人的に今のフロントエンドの流行を押さえつつ、プロダクトでの採用を考えた際に十分メリットを享受して採用できるレベルの技術を中心にまとめています。
Reactに限らず採用できる技術についても軽く触れるので、フロントエンドやTypeScriptに興味がある人にも参考になるかと思います。  

Webフロントエンドの歴史については以前の記事で整理していますので、興味がある方は参考にしてください。

[フロントエンド入門 フロントエンドの歴史](https://www.lyricrime.com/posts/frontend-history-2024/)

## Reactとは

Facebookが開発したJavaScriptライブラリで、現在はオープンソースとして公開されています。  
主にUIを構築するために使用され、コンポーネント指向のライブラリです。  

ほかにもVueやAngularなどのライブラリやフレームワークがありますが、Reactはその中でも特に人気が高いです。  
背景としては、Reactはリリースされてからある程度の年月が経過しており、ある程度枯れているため、安定している点や、対抗ライブラリであるVueが2.0から3.0へのアップデートで大幅な変更があり、追従に疲弊した開発者がReactを選択したケースも多いです。

個人的には、Vueも素晴らしい技術で、Reactと比較しても優れている点は多いと思います。  
ただ、Reactとのコミュニティの方向性の違いとして、Vue.jsはメジャーバージョンアップで大幅に変更があり、過去の負債を抱えずに理想的な形で進化していくことを目指しているのに対し、Reactはバージョンアップでの大幅な変更は少なく、安定性を重視しているという違いがあります。  

技術としてはReactでできることは大抵Vueでもできるので、この記事のスタンスとしてはどちらの技術を採用すべきかという点には言及しません。  

参考: [React - 2023年度リクルート エンジニアコース新人研修の講義資料です](https://speakerdeck.com/recruitengineers/react-2023)

## Reactの特徴

### コンポーネント志向

Reactはコンポーネント志向のライブラリです。
コンポーネントはUIの独立した部品であり、再利用可能なコードブロックとして扱うことができます。
これにより、複雑なUIを小さなパーツに分割し、管理しやすくなります。

例: Buttonコンポーネント

```jsx
import React from 'react';

function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}

export default Button;
```

上記の例では、`Button`というコンポーネントを定義しています。  
Reactでは、JSXという記法を使ってコンポーネントを記述します。これは、JavaScript + XMLの略で、HTMLのような記法でコンポーネントを記述することができます。  

2024現在のフロントエンドでは、TypeScriptを利用して開発することが主流となっており、JSXとTypeScriptを組み合わせてTSXという記法でコンポーネントを記述することが多いです。

例: Buttonコンポーネント（TypeScript）

```tsx
import React from 'react';

type ButtonProps = {
  label: string;
  onClick: () => void;
};

const Button = ({ label, onClick }: ButtonProps) => {
  return <button type="button" onClick={onClick}>{label}</button>;
};

export default Button;
```

### データの流れ(単方向データフロー)

Reactでは、ステート（state）とプロップス（props）という概念を使ってデータの流れを管理します。  
ステート(State): コンポーネント内部で管理される動的なデータ。ユーザーの操作やAPIからのデータ取得など、変化するデータを管理します。
プロップス(Props): 親コンポーネントから子コンポーネントに渡されるデータ。コンポーネント間でデータを受け渡すために使用します。

Reactでは、Propsを利用して上位の親コンポーネントが下位の子コンポーネントにデータを渡すのが基本的なデータの流れです。  

単方向データフローと useStateを組み合わせてみましょう。
useState フックを使ってコンポーネントの状態（state）を管理すると、その状態は変更が必要な特定のコンポーネント内にのみ存在します。状態を持っているコンポーネントが、そのデータを他のコンポーネントに渡すには props を使用します。

```tsx
import { useState } from 'react';

// 子コンポーネント: 表示とボタンを含む
type CounterDisplayProps = {
  count: number;
  increment: () => void;
};

const CounterDisplay = ({ count, increment }: CounterDisplayProps) => {
  return (
    <div>
      <p>カウント: {count}</p>
      <button type="button" onClick={increment}>
        インクリメント
      </button>
    </div>
  );
};

// 親コンポーネント: 状態を管理し、子コンポーネントに渡す
const CounterContainer = () => {
  const [count, setCount] = useState(0);

  // 子コンポーネントに渡す関数
  const incrementCount = () => setCount(prevCount => prevCount + 1);

  return <CounterDisplay count={count} increment={incrementCount} />;
};

export default CounterContainer;

```

親コンポーネント (CounterContainer) が状態 (count) を管理します。この状態は useState フックを使って定義されています。

親コンポーネントは状態を子コンポーネントにpropsを通して渡します。この例では、count と increment 関数が CounterDisplay に props として渡されています。

子コンポーネント (CounterDisplay) は渡された count を表示し、increment ボタンのクリックイベントに対応します。このように、状態管理を親コンポーネントに任せることで、子コンポーネントは「状態の表示」と「クリックイベントのハンドリング」という役割に専念できます。

この単方向データフローを利用することで、データの流れがシンプルになり、加えて親コンポーネントがデータを管理し、子コンポーネントがそれを表示するという役割分担が明確になります。このようなパターンは、デザインパターンとしてPresentational and Containerと名前がつけられています。  

### 宣言的UI

Reactは宣言的UIを採用しています。これは、「UIがどうあるべきか」を宣言することで、状態の変化に応じてUIが自動的に更新される仕組みです。
jQueryなどの命令型のUIライブラリ(手動でDOMを操作する方法)と比べて以下のメリットがあります。

- 可読性の向上: コンポーネントがどのように見えるかを宣言するため、コードが直感的になります。
- 保守性の向上: 状態管理が一元化され、バグの発生を抑えることができます。
- 再利用性の向上: コンポーネント志向とも被りますが、コンポーネントを再利用することが容易になります。

このブログ内でも記事を作成していますので参考にしてください。

[フロントエンド入門 宣言的UIと命令的UI](https://www.lyricrime.com/posts/declarative-ui/)

## Reactを取り巻くエコシステム

Reactはリリースから十分な時間が経ち、エコシステムも充実しています。  
以下にReactを取り巻く主なライブラリやツールのなかで、個人的なおすすめを紹介します。  

### SWR

SWRは先述したNext.jsの開発元であるVercelが開発したデータフェッチングライブラリです。  
キャッシュ、リフェッチ、再取得などの機能を提供し、React Queryと並んで人気のあるライブラリです。  
React Queryとの違いは、よりシンプルなAPIと、データ取得に特化している点でしょうか。

基本的な考え方はHTTPのキャッシュ戦略であるStale-While-Revalidateを採用しており、キャッシュが古い場合に古いデータを返しつつ、バックグラウンドで新しいデータを取得し、新しいデータが取得できたら古いデータを新しいデータに置き換えるというものです。

基本的な使い方

```ts
import useSWR from 'swr';

type User = {
  id: number;
  name: string;
};

const fetcher = (url: string): Promise<User[]> => fetch(url).then((res) => res.json());

function UserList() {
  const { data, error } = useSWR<User[]>('/api/users', fetcher);

  if (error) return <div>Failed to load</div>;
  if (!data) return <div>Loading...</div>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default UserList;
```

上記のように、`useSWR`フックを使用してデータ取得を行います。`useSWR`は第一引数として、キーを指定します。一般的にはURLのようなかたちでキーを指定することが多いですが、エンドポイントと直接関係はなく、あくまで一意なキーとして扱われます。  

```ts
const { data, error } = useSWR(userId ? `/api/users/${userId}` : null, fetcher);
```

のように動的なキーを指定することもできます。

useSWRからはデータの状態である`data`とエラーの状態である`error`を取得することができます。

### Zod

Zodはスキーマベースの型バリデーションライブラリで、TypeScriptとシームレスに統合できます。  
Zodを利用すると、オブジェクトや配列に対して、型チェックとバリデーションを同時に行うことができます。

特にバックエンドでTypeScriptを利用するような場合にサーバーとクライアントのデータチェックを統一できるのが特徴で、バックエンドとフロントエンドの整合性を保つ際に便利です。

```ts
import { z } from 'zod';

const userSchema = z.object({
  name: z.string().min(1, "名前は必須です"),
  email: z.string().email("有効なメールアドレスを入力してください"),
});

const result = userSchema.safeParse({ name: "", email: "invalid-email" });

if (!result.success) {
  console.log(result.error.errors);
}
```

上記の例では、`userSchema`というスキーマを定義し、`name`と`email`を持っています。  
nameは1文字以上であること、emailは有効なメールアドレスであることをバリデーションしています。  

`safeParse`メソッドを使うことで、バリデーションを行い、エラーがある場合はエラーメッセージを取得することができます。

- resultにはsuccessプロパティが含まれ、バリデーションが成功したかどうかを示します。
- result.successがfalseの場合、バリデーションエラーが発生していることを意味します。
- result.error.errorsにはエラー内容が格納されており、どのフィールドがどのような理由で不正であるかが詳細に記述されています。

また、このparseに成功した時点で、スキーマから生成される型に対して推論が効くため、レスポンス等をバリデーションしつつ型付けを行うことができます。

### React Hook Form

Reactでフォームを簡単に管理できるライブラリとして、React Hook Formがあります。  
シンプルで、パフォーマンスも良いため、Reactでフォームを扱う際にはおすすめです。  

React Hook Formでは、useFormという関数を使用してフォームの状態やバリデーションを管理します。

また、先に紹介したZodと組み合わせることで、フォームのバリデーションを型安全に行うことができます。

```ts
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const userSchema = z.object({
  name: z.string().min(1, "名前は必須です"),
  email: z.string().email("有効なメールアドレスを入力してください"),
});

type UserFormInputs = z.infer<typeof userSchema>;

function UserForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<UserFormInputs>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit = (data: UserFormInputs) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name")} />
      {errors.name && <span>{errors.name.message}</span>}
      
      <input {...register("email")} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

1. バリデーション: 上記の例だと、zodで定義したスキーマをresolverに指定しています。これでzodのスキーマがフォームに適用されます。
2. register: フォームの入力フィールドを`<input {...register("name")} />`のように登録します。これにより、フォームの状態が管理されます。
3. エラーメッセージの表示: バリデーションに失敗した場合、エラー情報がformState.errorsに格納されます。たとえば、nameフィールドが空の場合、errors.nameにエラーが設定され、そのエラーメッセージが{errors.name.message}に表示されます。
4. handleSubmitは、フォームの送信時にバリデーションを実行し、エラーがなければonSubmit関数にデータを渡します。

その他の特徴としては、React Hook Formは入力ごとに際レンダリングを行わないので、愚直なReactでのフォーム管理よりもパフォーマンスが向上します。

## フレームワーク

Reactを利用する際には、フレームワークと合わせて利用することをおすすめします。

### Next.jsとの組み合わせ

Next.jsはReactをベースにしたフレームワークで、サーバーサイドレンダリング（SSR）、静的サイト生成（SSG）、APIルートのサポートなどの機能を持ちます。

Next.jsはフルスタックフレームワークではありますが、バックエンドとフロントエンドを分離することができるため、バックエンドにLaravelやRuby on Railsといった従来のREST APIを使うこともできます。  

主な特徴

- ファイルベースのルーティング: pagesディレクトリに配置したファイルが自動的にルートとして認識されます。
- データフェッチング: getServerSidePropsやgetStaticPropsを使用して、サーバーサイドでデータを取得できます。
- APIルート: pages/apiディレクトリ内にAPIエンドポイントを簡単に作成できます。

[Next.js公式ドキュメント](https://nextjs.org/)

この記事で説明するにはあまりにも広範囲なため、詳細は公式ドキュメントや他のブログ等を参照してください。  

個人的な主観としては、2024年時点でReactを使うフレームワークとしては最も人気があるかなと思います。  
とはいえ、App Routerによるキャッシュ周りの複雑性やNext.js15での方向の転換などいくつか方向性が変わる可能性があるため、最新の情報を確認することをおすすめします。  

Next.jsを利用する場合は、Next.jsの思想に合わせていく覚悟が必要だと感じています。

### Remix

RemixはNext.jsと同じくReactをベースにしたフレームワークで、サーバーサイドレンダリング（SSR）、静的サイト生成（SSG）、APIルートのサポートなどの機能を持ちます。  

Next.jsに迫る勢いで人気が出てきているフレームワークで、Next.jsとは対象的に、Web標準API、つまりMDNに載っているAPIをほとんどラップせずに利用しているのが特徴です。  
公式ドキュメントにも、「Web標準と最新のウェブアプリUXに焦点を当てる」というような記載があります。

Remixで取り扱う概念のほとんどは、Webの歴史が作ってきた標準的な概念を利用しているため、フレームワーク独自の概念やAPIを覚える必要がなく、Webの知識がそのまま活かせるというメリットがあります。

過去フレームワーク固有の仕組みに依存して、破壊的変更等によって疲弊した開発者にとっては、Remixは魅力的な選択肢となるかもしれません。

- [Remix入門: フロントエンドもバックエンドも爆速開発を実現する次世代Webフレームワーク](https://zenn.dev/acompany/articles/123c29f46d213c)
- [Remix公式](https://remix.run/)

記事執筆時点でのフレームワークの開発の方向性としては、RemixはReact Routerと統合されつつあり、React Server ComponentsベースのRemixの開発が進んでいるようです。

### tRPC

tRPCは、TypeScriptのプロジェクトでバックエンドとフロントエンド間の型安全な通信を可能にするライブラリです。

簡単に言ってしまえばHTTP通信のラッパーですが、tRPCを使うと、APIのエンドポイントを定義してからそのレスポンスやリクエストに対して型を設定するのではなく、TypeScriptの型定義を直接共有しながら開発を進めることができます。

入門記事は以前書いたので、興味がある方は参考にしてください。

- [tRPCへの入門](https://www.lyricrime.com/posts/trpc-init/)

小中規模の、バックエンドにもTypeScriptを採用するようなプロジェクトでの採用は非常に効果的だと感じています。

ここまで紹介した技術の多くはT3 Stackと呼ばれる開発スタックにも採用され、TypeScriptでフルスタック開発を行う際に人気のある選択肢となっています。

- [Create T3 App](https://create.t3.gg/en/introduction)

## テストの導入

ユニットテストについては、まずはJestかVitestの採用を検討することになるかなと思います。

現状Jestでのテスト環境がない場合はVitestを採用すべきだと思っています。

理由は、JestはJavaScript前提で作成されており、歴史的にもNode.jsの流行やCJSとESMの混在等が問題になっていた時代に作られたライブラリであるため、現代のフロントエンドでESM、TypeScriptを使用したコードに対して、テストを書く場合はVitestの方が困るポイントが少ないと感じています。

VitestでReactのコードに対してテストを書くときの例を示します。

```ts
// Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';
import { vi } from 'vitest';

test('ボタンのクリックイベントが発火する', () => {
  const handleClick = vi.fn();
  render(<Button label="Click me" onClick={handleClick} />);
  
  fireEvent.click(screen.getByText(/click me/i));
  
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

もちろん、ロジック部分に対してもモック等を利用したりしつつテストを書くことも可能です。

担当しているプロダクトでは、フロントエンドにユニットテストを導入する際にJestを採用するかVitestを採用するかを検討しましたが、Viteに移行 + Vitestの採用の方がJestに比べると辛さが少ないと感じたため、Vitestを採用することにしました。  

参考: [Vite は使ってないけど Jest を Vitest に移行する](https://zenn.dev/sa2knight/articles/migrating_vitest_from_jest)

## ディレクトリ構造

フロントエンドにおけるディレクトリ構造においては、複雑さと向き合うために、ドメイン駆動設計のエッセンスを取り入れているところが多い印象を受けています。  
以前はAtomic Designなどが流行しましたが、必要以上に複雑になることが多いため、最近ではシンプルなディレクトリ構造を採用することが多いです。

ただ、ドメイン駆動設計を採用するとしてもComponentやPageといったフロントエンド固有のものやReactのHooksなどのファイルをどのように扱うかは、プロジェクトによって異なるのかなと思います。

## Formatterと静的解析

LinterやFormatterについては、ESlintとPrettierの組み合わせが主流です。最近ではこれらを統合したようなツールとして、Biomeというものが登場しています。

- ESlint + Prettier

現状のデファクトスタンダードであり、ほとんどのプロジェクトで採用されていると思います。  
ただ、設定の競合が発生したり、設定自体が複雑で、設定内容についてはプロジェクトごとに議論が必要となります。

- Biome

ESlintとPrettierを統合したようなツールで、単一のツールで静的解析とフォーマットを行うことができるため、設定が簡単であったり、パフォーマンスが高いことが特徴です。  
ただ、ESlintやPrettierのようにプラグインエコシステムがないため、カスタマイズ性は低くなります。

個人的にはあまり静的解析やフォーマットにこだわりがなく、カスタマイズも最小限に抑えるべきだと考えているため、Biomeを採用することが多いです。

## Reactの最新機能とトレンド

ここからは2024年現在のReactの最新機能について紹介します。  
どれくらい主流になるかは未知数なので、プロダクトへの本採用を検討する場合は慎重に検討することをおすすめします。  

### React Server Components

React Server Components（RSC）は、サーバーサイドでコンポーネントをレンダリングし、クライアントに必要な部分だけを送信することで、パフォーマンスを最適化する新しいアーキテクチャです。

サーバーコンポーネントを利用すると、以下のようなメリットがあります。

サーバーサイドレンダリングの効率化: RSCは、サーバー上でコンポーネントをレンダリングし、クライアントには軽量なJavaScriptとして送信されます。これにより、初期ロード時間が短縮されます。

クライアントとサーバーのコード分離: クライアント専用のコードとサーバー専用のコードを明確に分離できます。また、セキュリティ面でも有利です。

データフェッチングの簡素化: サーバー上でデータを取得し、コンポーネントに直接渡すことができるため、クライアント側でのデータ管理が簡素化されます。

```tsx
// ServerComponent.server.tsx
import React from 'react';

type User = {
  id: number;
  name: string;
};

const fetchUsers = async (): Promise<User[]> => {
  const response = await fetch('https://api.example.com/users');
  return response.json();
};

const ServerComponent = async () => {
  const users = await fetchUsers();
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

export default ServerComponent;

```

こうしてみてみると、ややRemixにも似ている気がしますね。

### Suspense

Suspenseは、非同期操作（例えばデータフェッチングやコードスプリッティング）を簡潔に扱うための仕組みです。Reactの描画プロセスを一時停止し、必要なデータが揃うまで待機することでロード中の状態を管理します。  

非同期データの扱いやすさ: Suspenseを使用することで、データが揃うまでコンポーネントのレンダリングを待機させ、ロード中の状態を簡単に管理できます。

コードスプリッティングの簡素化: 大規模なアプリケーションにおいて、必要な部分だけを遅延ロードすることで、初期ロード時間を短縮できます。

```tsx

import React, { Suspense } from 'react';

const LazyComponent = React.lazy(() => import('./LazyComponent'));

const App = () => (
  <div>
    <h1>Welcome to React</h1>
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  </div>
);

export default App;

```

## まとめ

2024年現在、もっとも主流なフロントエンドのライブラリはReactであると言えます。  
その背景には、Reactの安定性やコミュニティの活発さ、豊富なエコシステムなどが挙げられます。  
Reactをベースに簡単に、UXの向上やパフォーマンスの最適化を行うためのライブラリやツールも多く存在しており、フロントエンド開発においてReactを採用することは非常に有益であると言えます。  

LaravelやRuby on RailsといったフルスタックフレームワークからReactベースに移行する場合にはルーティングやAPIの設計などが課題となることが多いですが、それを差し引いてもReactを採用するメリットは大きいと感じています。

## 今回の記事での参考資料

- [フロントエンド入門 フロントエンドの歴史](https://www.lyricrime.com/posts/frontend-history-2024/)
- [React - 2023年度リクルート エンジニアコース新人研修の講義資料です](https://speakerdeck.com/recruitengineers/react-2023)
- [フロントエンド入門 宣言的UIと命令的UI](https://www.lyricrime.com/posts/declarative-ui/)
- [Next.js公式ドキュメント](https://nextjs.org/)
- [Remix入門: フロントエンドもバックエンドも爆速開発を実現する次世代Webフレームワーク](https://zenn.dev/acompany/articles/123c29f46d213c)
- [Remix公式](https://remix.run/)
- [tRPCへの入門](https://www.lyricrime.com/posts/trpc-init/)
- [Create T3 App](https://create.t3.gg/en/introduction)
- [Vite は使ってないけど Jest を Vitest に移行する](https://zenn.dev/sa2knight/articles/migrating_vitest_from_jest)