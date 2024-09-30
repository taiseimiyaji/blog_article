---
title: フロントエンドの状態管理について調べてみる
tags: [frontend]
createDate: 2024-09-30
updateDate: 2024-09-30
slug: frontend-state
draft: false
---

## はじめに

今回はフロントエンドの状態管理について調査してみます。

## フロントエンドにおける状態管理とは

フロントエンドにおける状態管理とは、ユーザーインターフェース（UI）のデータやその変化を一貫性を持って管理する手法を指します。  
ここで言う「状態（State）」とは、サーバーから取得したデータ、UIの現在の表示内容など、アプリケーションの動作に影響を与える全ての情報を意味します。  
ユーザーからの入力も状態に含まれます。

今回は自分の考えにも近いこちらの記事を参考に状態管理周りを調査してみます。

[2020年に立ち上げたWebフロントエンド構成の振り返り](https://zenn.dev/knowledgework/articles/32371c83e68cbe)

## 状態が必要な理由

状態管理を行わない、もしくは不適切な状態管理を行なった場合のつらさを確認してみます。

### データの不整合  

主流となっている宣言的UI系のライブラリ、Vue.jsやReactでは、コンポーネントの再利用性を高めるために、コンポーネントごとにデータを管理することができます。

この管理をした場合、各コンポーネントが独自の状態を持つことになります。これにより、同じリソースを表示すべき箇所が複数ある場合はUIコンポーネント間でデータの不整合が発生する可能性があります。  

### コードの複雑化

データの受け渡しの流れを適切に整理せず、コンポーネント間でデータを受け渡すことが多くなると、コードの複雑化が進みます。  
UIはどうしても親子関係が入れ子になるため、親から子へのデータの受け渡し、子から親へのデータの受け渡し、兄弟間でのデータの受け渡しなど、データの流れを把握するのが難しくなります。  

後述するVuexの公式ドキュメントには下記の記載があります。

>しかし、単純さは、共通の状態を共有する複数のコンポーネントを持ったときに、すぐに破綻します:  
複数のビューが同じ状態に依存することがあります。  
異なるビューからのアクションで、同じ状態を変更する必要があります。  
一つ目は、プロパティ (props) として深く入れ子になったコンポーネントに渡すのは面倒で、兄弟コンポーネントでは単純に機能しません。二つ目は、親子のインスタンスを直接参照したり、イベントを介して複数の状態のコピーを変更、同期することを試みるソリューションに頼っていることがよくあります。これらのパターンは、いずれも脆く、すぐにメンテナンスが困難なコードに繋がります。  
では、コンポーネントから共有している状態を抽出し、それをグローバルシングルトンで管理するのはどうでしょうか？ これにより、コンポーネントツリーは大きな "ビュー" となり、どのコンポーネントもツリー内のどこにあっても状態にアクセスしたり、アクションをトリガーできます!  
さらに、状態管理に関わる概念を定義、分離し、特定のルールを敷くことで、コードの構造と保守性を向上させることができます。  
これが Vuex の背景にある基本的なアイディアであり、Flux、 Redux そして The Elm Architectureから影響を受けています。 他のパターンと異なるのは、Vuex は効率的な更新のために、Vue.js の粒度の細かいリアクティビティシステムを利用するよう特別に調整して実装されたライブラリだということです。  

### バグの増加と開発効率の低下

機能追加時や変更時に状態の管理が複雑になってしまうと、理解するのに時間がかかり、開発スピードが落ちます。  
同じデータを扱う場合の同期の問題など、データのスコープを適切に設計しないと、バグが発生しやすくなります。

## Vue.jsにおける状態管理

Vue.js 2.xでは、Vuexという状態管理ライブラリが提供されています。現在のVue.js 3.xでもVuexは引き続き利用可能ですが、Vue 3.xではComposition APIを利用することで、Vuexを使わずに状態管理を行うことも可能です。似たような状態管理ライブラリとして、Piniaが公式で推奨されています。

### Vuex

Vuexはアプリケーション全体で共有される集中型ストアを持ちます。これはFluxというアーキテクチャにインスパイアされており、他のライブラリにも影響を与えています。

VUexは以下の要素で構成されています。

- State：アプリケーションの状態を保持するオブジェクト。
- Getters：状態を取得するための算出プロパティ。
- Mutations：状態を変更するためのメソッド。同期的な処理。
- Actions：ミューテーションをコミットするためのメソッド。非同期処理も可能。

Stateの定義の仕方

``` javascript
const store = new Vuex.Store({
  state: {
    count: 0
  }
})
```

Stateの取得、Gettersの定義の仕方

``` javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  getters: {
    doubleCount(state) {
      return state.count * 2
    }
  }
})
```

Mutationの定義の仕方

``` javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    }
  }
})
```

Actionでの非同期処理の定義の仕方

``` javascript
const store = new Vuex.Store({
  state: { count: 0 },
  mutations: { /* ... */ },
  actions: {
    incrementAsync({ commit }) {
      setTimeout(() => {
        commit('increment');
      }, 1000);
    }
  }
});
```

### Fluxとは

FluxはFacebookが提唱するアプリケーションのアーキテクチャです。  
Fluxは3つの要素から構成されています。  

| 要素 | 説明 |
| --- | --- |
| Store | アプリケーションの状態を保持するオブジェクト、および状態の更新処理 |
| Action | ユーザーの操作やAPIのレスポンスなど、アプリケーションの状態を変更するための処理 |
| Dispatcher | Storeに対してActionを発火させる |

Fluxの特徴として、データの流れが一方向であることが挙げられます。

流れとしてはAction -> Dispatcher -> Store -> Viewとなります。

状態を更新する場合は

1. ViewからActionを発火(ボタンを押す、文字を入力するなど)
2. 更新したい内容をActionとしてDispatcherに送信
3. DispatcherがStoreにActionを送信
4. Storeの状態がActionの内容に応じて更新され、それを検知したViewが再描画される

という流れになります。

この方法によって、データの流れを逆流させたり、Dispatcherを経由せずにStoreを直接更新することができないため、データの一貫性を保つことができます。

ちなみに現在主流となっているPiniaやVuex5と呼ばれるものはFluxアーキテクチャをベースにしてはいますが、よりシンプルでFluxアーキテクチャを理解する必要がないように設計されています。

## Reactにおける状態管理

ReactにおいてもFluxアーキテクチャをベースにした状態管理ライブラリがいくつか存在します。  
Reduxが有名ですが、他にもMobXやRecoil、Zustandなどがあります。  

ですが、今回はそれらのライブラリの解説ではなく、自分の考えている状態管理の最適解についてまとめてみます。

## 状態の分類

参考記事にもある通り、フロントエンドで扱う`状態`は下記のように分類できると考えます。

- サーバーから取得したデータのキャッシュとしての状態
- アプリケーション全体で持つグローバルな状態
- コンポーネント内でのみ持つローカルな状態

## サーバーから取得したデータのキャッシュとしての状態

### SWR

SWR(Stale-While-Revalidate)は、Next.jsの開発元であるVercel社が提供するReact Hooksライブラリです。  
主な責務はデータの取得とキャッシュを簡単に行うことです。

SWRは、HTTPのキャッシュ戦略であるStale-While-Revalidateを採用しています。([MDN web docs](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Cache-Control))

この戦略は、キャッシュが古い場合に古いデータを返しつつ、バックグラウンドで新しいデータを取得し、新しいデータが取得できたら古いデータを新しいデータに置き換えるというものです。

基本的な使い方は下記の通りです。

``` javascript
import useSWR from 'swr';

const fetcher = url => fetch(url).then(res => res.json());

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher);

  if (error) return <div>Failed to load</div>;
  if (!data) return <div>Loading...</div>;

  return <div>Hello {data.name}!</div>;
}
```

特徴としては、下記のような点が挙げられます。

- キャッシュの有効期限を設定できる
- フォーカス時や再接続時にデータを再取得する
- `api/user`のような部分はエンドポイントではなくキー。同じキーを指定することで、複数のコンポーネントでデータを共有できる

ただ、あくまでデータフェッチに特化しており、サーバーからのデータ取得に特化しているため、状態管理ライブラリとしては不十分です。

### React Query

React Queryは、Reactアプリケーションでデータを取得、キャッシュ、更新、削除するためのライブラリです。  
SWRと比較すると、より高度な機能を持っています。  

もともと、React QueryやSWRを利用しない場合はuseStateとuseEffectを組み合わせてデータの取得とキャッシュを行っていましたが、React Queryを利用することで、データの取得やキャッシュを簡単に行うことができます。

初期設定

``` javascript
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { QueryClient, QueryClientProvider } from 'react-query';
import App from './App';

const queryClient = new QueryClient();

ReactDOM.render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>,
  document.getElementById('root')
);

```

データの取得

```javascript
// App.js
import React from 'react';
import { useQuery } from 'react-query';

function fetchUsers() {
  return fetch('https://api.example.com/users').then(res => res.json());
}

function App() {
  const { data, error, isLoading } = useQuery('users', fetchUsers);

  if (isLoading) return <div>読み込み中...</div>;
  if (error) return <div>エラーが発生しました: {error.message}</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default App;

```

useQueryの第一引数にはキーを指定します。このキーはデータのキャッシュに利用されます。  
useQueryの第二引数にはデータを取得する関数を指定します。この関数は非同期関数である必要があります。  
その他状態としてisLoadingやerrorが返却されるため、それに応じて表示を変更することができます。  

React Queryはデフォルトでデータをキャッシュし、一定時間が経過すると自動的に再フェッチします。これにより、データの新鮮性を保ちながら、不要なリクエストを削減できます。  

キャッシュの有効期限を設定することも可能です。

```javascript
useQuery('users', fetchUsers, {
  staleTime: 1000 * 60 * 5, // 5分間データを新鮮とみなす
});
```

取得だけでなく、データの更新や削除も簡単に行うことができます。

```javascript
import { useMutation, useQueryClient } from 'react-query';

function AddUser() {
  const queryClient = useQueryClient();

  const mutation = useMutation(newUserData => {
    return fetch('https://api.example.com/users', {
      method: 'POST',
      body: JSON.stringify(newUserData),
    });
  }, {
    onSuccess: () => {
      // 'users'クエリを無効化して再フェッチ
      queryClient.invalidateQueries('users');
    },
  });

  const handleAddUser = () => {
    mutation.mutate({ name: '新しいユーザー' });
  };

  return (
    <button onClick={handleAddUser}>
      ユーザーを追加
    </button>
  );
}
```

ちなみにSWRにもuseSWRMutationというデータの更新を行うためのフックが用意されていますが、React Queryの方がより高度な機能を持っています。  
シンプルさを求める場合はSWR、より高度な機能を求める場合はReact Queryを利用すると良いでしょう。  

## アプリケーション全体で持つグローバルな状態

これはSWRやReact Queryでは対応できないため、状態管理ライブラリを利用する必要があります。  
とはいえコンポーネント内で保持できるものは後述するローカルな状態で十分であるため、アプリケーション全体で持つグローバルな状態を持つ必要がある場合は、VuexやRedux等ライブラリを利用して、状態管理を行うことが一般的です。  

例えば、認証の情報やサイドバーの開閉状態、検索条件などページを跨いでも保持しておきたい情報がこれに該当します。

参考記事ではRecoilを利用していますが、こちらに関しては触ったことがないため、ここでは割愛します。

この記事も参考になりそうです。
[Facebook製の新しいステート管理ライブラリ「Recoil」を最速で理解する](https://blog.uhy.ooo/entry/2020-05-16/recoil-first-impression/)

## コンポーネント内でのみ持つローカルな状態

最後に、コンポーネント内で保持するローカルな状態についてです。

ページを跨いで保持する必要のない状態はReactの場合はuseStateやuseReducerを利用して管理することができます。  
単純である代わりにテストがしづらいことが参考記事では挙げられていますが、storybookを利用したフロントエンドのテストは現状取り組めていないため、今後の課題としたいと思います。

## まとめ

フロントエンドの状態管理について調査してみました。  
状態管理の悩みの種であるサーバーからのデータの取得とキャッシュについてを中心に調査しましたが、React QueryやSWRを利用することで、簡単にデータの取得とキャッシュを行うことができることがわかりました。  
コンポーネントのテストや、アプリケーション全体で共有する状態については、まだまだ課題が残っているため、今後も引き続き調査を行っていきたいと思います。
