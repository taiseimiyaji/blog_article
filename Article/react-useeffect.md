---
title: ReactのuseEffectをきちんと理解する
tags: [react, useEffect]
createDate: 2023-06-26
updateDate: 2022-06-26
slug: react-use-effect
---

## はじめに

最近ReactやNext.jsを触る機会が多いのですが、スマレジでの業務で使用しているわけではないため、キャッチアップがおろそかになっているなあと感じています。  
今回は、Reactに触れる上で必須かつVueとの違いにおいて重要な仕組みであるuseEffectについて公式サイトを確認してみます。  

## useEffectとは何か

公式サイトにあるuseEffectの呼び出し方は以下のようになっています。

```jsx
useEffect(setup, dependencies?)
```

ReactのuseEffectは、コンポーネントのライフサイクルを扱うための重要なフックです。副作用とは、データの取得、購読の設定、手動でのReactコンポーネントのDOMの変更など、Reactのレンダリングに影響を与える可能性のある操作のことを指します。useEffectは以下のように使用します。

```jsx
useEffect(() => {
  // 副作用を実行するコード
}, [/* 依存配列 */]);
```

**Tip:** useEffectはコンポーネントのレンダリング後に実行されます。これにより、副作用がコンポーネントの更新をブロックすることなく、非同期に実行されます。

## 依存配列とは何か

useEffectの第二引数には依存配列が渡されます。この配列は、useEffect内の副作用が依存する値のリストです。依存配列の値が変更されると、再度useEffectの第一引数に指定した副作用が実行されます。

```jsx
useEffect(() => {
  console.log(`Count is now ${count}`);
}, [count]);
```

この例では、`count`が変更されるたびに、副作用が再度実行されます。

**Tip:** 依存配列を省略すると、副作用はすべてのレンダリング後に実行されます。これは、副作用が値の変更に関係なく常に実行されるべき場合に便利です。

## 副作用のクリーンアップ

副作用は、イベントリスナーの追加など、クリーンアップが必要な操作を含むことがあります。useEffectは、副作用の関数がクリーンアップ関数を返すことが可能です。

```jsx
useEffect(() => {
  const subscription = someSubscribeFunction();

  return () => {
    someUnsubscribeFunction(subscription);
  };
}, [/* 依存配列 */]);
```

この例では、副作用の関数が購読(Subscribe)を設定し、クリーンアップ関数がその購読を解除します。クリーンアップ関数は、コンポーネントがアンマウントされた時や、依存配列の値が変更される前に実行されます。

**Tip:** クリーンアップ関数は、副作用が次に実行される前にも呼び出されます。この時、古い値を使用してクリーンアップ関数が実行され、そのあとに新しい値を使用してセットアップ関数が実行されます。

公式サイトでは、第一引数に渡した関数をセットアップ関数、返される関数をクリーンアップ関数と呼んでいます。

なお、useEffectを使用する際には、副作用内で使用されるすべてのステートやプロップを依存配列に含めることが推奨されています。

また、useEffect内で非同期操作を行う場合、直接async関数を渡すことはできません。代わりに、即時関数を使用します。

```jsx
useEffect(() => {
  (async () => {
    const data = await fetchData();
    // Do something with data
  })();
}, [/* 依存配列 */]);
```

## 具体的なuseEffectの使用例

APIからデータを取得するという場合のuseEffectの使用例を考えてみます。

### APIからデータを取得する

Reactアプリケーションでは、外部APIからデータを取得することがよくあります。useEffectは、コンポーネントがマウントされた後にAPIからデータを取得するのに使用することができます。

以下に、useEffectを使用してAPIからデータを取得し、そのデータをコンポーネントのステートに保存する例を示します。

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios('https://api.example.com/data');
      setData(result.data);
    };

    fetchData();
  }, []); // 依存配列は空なので、レンダリング後に副作用は一度だけ実行されます。

  return (
    <div>
      {data ? data.map(item => (
        <div key={item.id}>{item.name}</div>
      )) : 'Loading...'}
    </div>
  );
}

export default App;
```

この例では、useEffectの中で非同期関数`fetchData`を定義し、その中でaxiosを使用してAPIからデータを取得しています。取得したデータは、`setData`を使用してステートに保存されます。そして、そのステートはコンポーネントのレンダリングで使用されます。

**Tip:** 非同期操作を行う場合、useEffectのコールバック関数は直接asyncにすることはできません。そのため、非同期関数を定義して即時に呼び出すというパターンを使用します。

ここまでが外部実行する場合の例ですが、公式サイトでは`外部システムと同期しようとしていない場合は、[useEffectは必要ない](https://react.dev/learn/you-might-not-need-an-effect)と明記されています。

### 使用する上での注意事項

- コンポーネントまたはカスタムフックの最上位でのみ使用することができる
- Reactには、バグの早期発見のために[Strict Mode](https://react.dev/reference/react/StrictMode)というものが存在します。これがオンの場合、開発中にはセットアップとクリーンアップサイクルを1回追加で実行されます。何度実行されても問題ないようにクリーンアップ関数を正しく実装するための措置です。
- 依存関係の一部がコンポーネント内で定義されたオブジェクトまたは関数である場合、エフェクトが必要以上に頻繁に再実行される危険性があります。これはコンポーネントがレンダリングされるたびに、新しいオブジェクトや関数が作成されるためです。これを回避するためには不要な依存性を排除し、非リアクティブなロジックは可能な限りuseEffectの外で実行する必要があります。
- useEffectはクライアント上でのみ動作するため、SSR時にはuseEffectは実行されません。

## useEffectのベストプラクティス

### 1. 依存配列を正しく使用する

useEffectの第二引数として渡される依存配列は、副作用が依存する値のリストです。この配列の値が変更されると、副作用は再度実行されます。

**ベストプラクティス:** 副作用内で参照されるすべてのステートとプロップを依存配列に含めます。これにより、副作用は常に最新のステートとプロップの値に基づいて実行されます。

**よくある間違い:** 依存配列を省略したり、不完全な依存配列を指定したりすると、副作用は期待したとおりに動作しない可能性があります。副作用が古いステートやプロップの値を参照することになり、バグの原因となる可能性があります。

### 2. 副作用で非同期操作を行う

useEffect内で非同期操作を行うことは一般的ですが、useEffectのコールバック関数は直接asyncにすることはできません。これは、useEffectのコールバック関数がクリーンアップ関数を返すことが期待されているためです。

**ベストプラクティス:** 先述した例同様、非同期操作を行う場合、useEffect内で非同期関数を定義し、その関数を即時に呼び出すことができます。

```jsx
useEffect(() => {
  const fetchData = async () => {
    const data = await someAsyncOperation();
    // Do something with data
  };

  fetchData();
}, [/* 依存配列 */]);
```

### 3. クリーンアップを忘れない

副作用は、購読の設定やイベントリスナーの追加など、クリーンアップが必要な操作を含むことがあります。useEffectは、副作用の関数がクリーンアップ関数を返すことで、これをサポートします。

**ベストプラクティス:** 副作用がクリーンアップを必要とする場合、副作用の関数からクリーンアップ関数を返します。このクリーンアップ関数は、コンポーネントのアンマウント時や、依存配列の値が変更される前に呼び出されます。

```jsx
useEffect(() => {
  const subscription = someSubscribeFunction();

  return () => {
    someUnsubscribeFunction(subscription);
  };
}, [/* 依存配列 */]);
```

### 4. useEffectとuseStateを組み合わせる

useEffectは、ステートの変更をトリガーとして副作用を実行するために、しばしばuseStateと組み合わせて使用されます。

**ベストプラクティス:** useStateで定義したステートを、useEffectの依存配列に含めます。これにより、ステートの変更が副作用の再実行をトリガーします。

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  document.title = `Count is ${count}`;
}, [count]);
```

**よくある間違い:** ステートの更新関数（この例では`setCount`）をuseEffectの依存配列に含める必要はありません。Reactはステートの更新関数のアイデンティティを安定させているため、これを依存配列に含めると不要な副作用の再実行が発生する可能性があります。

## useEffectを使ってAPIの取得の共通化を行う

### カスタムフックの作成

複雑なロジックを扱う場合、フックのコードは複雑になりがちです。この問題を解決するために、Reactはカスタムフックの概念を導入しました。

カスタムフックは、フックのロジックを再利用可能な関数に抽出することができます。カスタムフックは、`use`から始まる関数として定義できます。カスタムフック内では、他のフック（useState、useEffectなど）を使用することができます。

以下に、APIからデータを取得するためのカスタムフックの例を示します。

```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

function useFetchData(url) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(url);
      setData(result.data);
      setIsLoading(false);
    };

    fetchData();
  }, [url]);

  return { data, isLoading };
}

export default useFetchData;
```

この`useFetchData`カスタムフックは、指定されたURLからデータを取得し、そのデータとローディング状態を返します。

```jsx
import useFetchData from './useFetchData';

function App() {
  const { data, isLoading } = useFetchData('https://api.example.com/data');

  if (isLoading) {
    return 'Loading...';
  }

  return (
    <div>
      {data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

export default App;
```

## 参考

[useEffect](https://react.dev/reference/react/useEffect)

[Strict Mode](https://react.dev/reference/react/StrictMode)

## 所感

今回はuseEffectについてのドキュメントを読んでみましたが、useEffectの使い方についてはある程度理解していたつもりでしたが、useEffectのベストプラクティスについては知らないことが多かったので勉強になりました。  
ここ最近、自分自身が勉強していく中で知らないことが多すぎることをだんだんと認識しはじめ、体系的なキャッチアップが少しおざなりになっているなと感じていました。  
これはひとえにインプット量の少なさからきているものだと思うので公式ドキュメントや技術書を読み込む時間を習慣にしていければと思います。  
