---
title: JavaScriptにおける非同期処理について
tags: [test]
createDate: 2022-12-18
updateDate: 2022-12-18
slug: javascript-axios
---

## はじめに

今までの学習内容でフロントエンドにあまり触れてなかったので今回はJavaScriptについて改めて理解をしていこうと思います。  
JavaScriptに対する個人的な理解度はある程度文法や言語思想は理解できているもののJavaScriptらしい部分や歴史的な部分、いわゆるその言語の特性を理解できているとは言えないレベルだなと感じているため、今回は非同期処理について改めて調査してみます。


## 同期処理と非同期処理

多くのプログラミング言語には同期処理(sync)と非同期処理(async)の２つのコードの評価の仕方があります。
特にJavaScriptにおいて非同期処理は重要な概念になります。

### 同期処理

同期処理ではコードを順番に処理していき、ひとつの処理が終わるまで次の処理は行いません。  
同期処理においては実行している処理はひとつだけになり、直感的な動作となります。  
一方、同期的にブロックする処理が行われていた場合にはひとつの処理が終わるまで、次の処理へ進むことができないです。

このときに特に問題となるのはブラウザ上でJavaScriptを動作させる場合です。  
基本的にJavaScript派ブラウザのメインスレッド(UIスレッドとも呼ばれる)で実行されます。
メインスレッドは表示の更新といったUIに関する処理も行っています。
そのため、メインスレッドがJavaScriptの処理で専有されると、表示が更新されなくなり、見た目上フリーズしたようになります。

### 非同期処理

非同期処理はコードを順番に処理していきますが、ひとつの非同期処理が終わるのを待たずに次の処理を実行します。
つまり、同時に実行される処理が複数存在します。  
これによって解決される問題としては、コストが大きく異なる2つの処理を同期的に順次処理していく効率の悪さを解決できます。


ここまでの非同期処理の説明だけを見ると、完全に別々の処理が同時進行しているように感じます。
基本的にJavaScriptの基本的な非同期処理はメインスレッドで実行されています。
`setTimeOut`メソッドなどを利用すれば並列処理ができるのではないかと感じますが、実際には`setTimeout`で指定された作業は一旦脇に置かれているだけで、メインスレッド上で順番に処理されています。

例外としては[Web Worker](https://developer.mozilla.org/ja/docs/Web/API/Web_Workers_API/Using_web_workers) APIを使用した場合などです。

## スレッドとは

非同期処理について理解を進める前に`スレッド`について理解しておく必要があります。   
`スレッド`(thread)はプログラムが連続して順番に何かしらの処理が実行される流れのことです。   
英語で「糸」という意味があり、一般的なプログラムはこのスレッド(糸)を複数合わせて一本の丈夫なひも(機能)を実現しているとイメージするとわかりやすいかもしれません。   
JavaScriptは基本的にシングルスレッドである、というのはこのスレッドが基本的に1つしかないということです。

### JavaScriptにおけるメインスレッド

ブラウザにおいて、JavaScriptは以下の2つのしごとをメインスレッドで行っています。

- JavaScriptの処理実行
- 画面へのレンダリング(描写)処理


JavaScriptでは一部の例外を除き、非同期処理は**並行処理(concurrent)**として扱われます。
並行処理とは、処理を一定の単位ごとに分けて処理を切り替えながら実行することです。
非同期処理を実装すると、メインスレッドに並んでいる処理の流れから一旦外れて次の処理に実行を譲るイメージです。

一方、先程例外の例に上げた`Web Worker`におけるは**並列処理**です。
並列処理とは、排他的に複数の処理を同時に実行することです。
Web Workerではメインスレッドとは異なるWorkerスレッドで実行されるため、Workerスレッド内で同期的にブロックする処理を実行してもメインスレッドは影響を受けにくくなります。
これによって重たい処理をWorkerスレッドに移動できます。

このように、非同期処理をひとくくりにはできないですが、基本的には`JavaScriptはシングルスレッドで実行される`という性質を知っておくことが大事です。つまり、ここから先紹介する非同期処理の仕組みはほとんど**並行処理**となります。

## 非同期処理と例外処理

### JavaScriptにおける例外処理(同期処理)

JavaScriptの場合、同期処理では[try...catch](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/try...catch)構文を使用することで同期的に発生した例外がキャッチできます。

```JavaScript
try {
    throw new Error("同期的なエラー");
} catch (error) {
    console.log("同期的エラーをキャッチ");
}
console.log("この行は実行されます");
```

### JavaScriptにおける例外処理(非同期処理)

非同期処理ではtry...catchによる例外のキャッチができません。

```JavaScript
try {
    setTimeout(() => {
        throw new Error("非同期的なエラー");
    }, 10);
} catch (error) {
    console.log("実行されない");
}
console.log("この行は実行されます");
```

tryブロックはそのブロック内で発生した例外をキャッチする構文です。
しかし、`setTimeout`関数で登録されたコールバック関数が実際に実行されて例外を投げるのは、すべての同期処理が終わったあととなります。
つまり、tryブロックのマークしている範囲外で例外が発生するため、catchできないという仕組みです。

そのため、コールバック関数内で同期的なエラーとしてキャッチします。

```JavaScript
// 非同期処理の外
setTimeout(() => {
    // 非同期処理の中
    try {
        throw new Error("エラー");
    } catch (error) {
        console.log("エラーをキャッチできる");
    }
}, 10);
console.log("この行は実行されます");
```

上記のようにコールバック関数内でエラーのキャッチは可能ですが、非同期処理の外からは非同期処理の中で例外が発生したかがわかりません。
非同期処理の外から、例外が発生したことを知るためには非同期処理の外へ伝える方法が必要です。

また、JavaScriptでのHTTPリクエストやファイルの読み書きといった処理も非同期処理のAPIとして提供されているため、例外の扱い方は重要になります。

非同期処理で発生した例外の扱い方には様々なパターンがありますが、主流な`Promise`について見ていきます。

## Promise

非同期処理がいくつも連なる場合にコールバック関数を利用すると、入れ子が深くなりすぎて1つの関数が肥大化する傾向にあります。

```JavaScript
first(function(data) {
    console.log("最初に実行する処理");
    second(function(data) {
        console.log("first関数が成功した場合に実行する処理");
        third(function(data) {
            console.log("second関数が成功したときに実行する処理");
        });
    });
});

```
このような問題を解決するのが`Promise`オブジェクトの役割です。

これまで、jQueryやAngularJSには似たような機能を提供してきましたが、ES2015でPromiseオブジェクトが標準化されたことで外部ライブラリに頼る必要がなくなりました。

非同期処理はPromiseのインスタンスを返し、そのPromiseには状態変化をした際に呼び出されるコールバック関数を登録できます。

```JavaScript
function asyncProcess(value) {
    return new Promise((resolve, reject) => {
        // ここで非同期処理を行う
        setTimeout(() => {
            if (value) {
                // 成功した場合はresolveを呼ぶ
                resolve(`入力値: ${value}`);
            } else {
                // 失敗した場合はrejectを呼ぶ
                reject('入力は空です');
            }
        }, 500);
    });
}

asyncProcess('input').then(() => {
    // 非同期処理が成功したときの処理
}).catch(() => {
    // 非同期処理が失敗したときの処理
})
```

asyncProcess関数は`Promise`オブジェクトのインスタンスを返しています。
PromiseインスタンスはasyncProcess関数内で行われた非同期処理が成功したか失敗したかの状態を表すオブジェクトです。  
また、この`Promise`インスタンス二台した`then`や`catch`メソッドで成功時や失敗時に呼び出される処理をコールバック関数として登録することができます。

書き方だけを見るとややこしく見えますが、Promiseは非同期処理の状態や結果を監視するためのオブジェククトです。  
同期的な関数では関数を実行するとすぐに結果がわかりますが、非同期な関数では関数を実行してもすぐには結果がわからないため、非同期処理の状態をラップしたオブジェクトを返し、結果が決まったら登録しておいたコールバック関数へ結果を渡す仕組みになっています。


### 具体的にPromiseインスタンスを理解する
上記のコードで基本的な使用方法は把握できるかと思いますが今回はPromiseについてもう少し丁寧に理解してみます。

まずはPromiseインスタンスの作成します。  
thenメソッドでPromiseがresolve、rejectしたときに呼ばれるコールバック関数を登録します。

```JavaScript
const promise = new Promise((resolve, reject) => {
    // 非同期の処理が成功したときはresolve()を呼ぶ
    // 非同期の処理が失敗したときはreject()を呼ぶ
});
const onFulfilled = () => {
    console.log("resolve時に実行される");
};
const onRejected = () => {
    console.log("reject時に実行される");
};

promise.then(onFulfilled, onRejected);
```

### Promise.prototype.thenとPromise.prototype.catch

Promiseのthenメソッドは成功(`onFulfilled`)と失敗(`onRejected`)の２つのコールバック関数を受け取りますが、どちらの引数も省略できます。

```JavaScript

function Process(path) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (path.startsWith("/success")) {
                resolve({ body: `Response body of ${path}` });
            } else {
                reject(new Error("Not Found"));
            }
        }, 1000 * Math.random());
    });
}

// thenメソッドで成功時と失敗時のコールバック関数を登録
Process("/success/data").then(function onfulfilled(response) {
    console.log(response);
}, function onRejected(error) {
    console.log("実行されない");
});

Process("/failure/data").then(function onFulfilled(response) {
    console.log("実行されない");
}, function onRejected(error) {
    console.error(error); // "Not Found"
});

```

ここで、失敗時のコールバック関数のみ登録する場合を考えます。
このときcatchメソッドは内部的にthenメソッドを呼び出しています。つまり、catchはthenの失敗時のみを記載するためのエイリアスとして動作します。

参考:  
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch

```JavaScript
function errorProcess(message) {
    return new Promise((resolve, reject) => {
        reject(new Error(message));
    });
}

// 非推奨。thenで失敗時のコールバック関数のみを渡したい場合は第一引数にはundefinedを渡す。
errorProcess("thenでエラーハンドリング").then(undefined, (error) => {
    console.log(error.message);
});

// 推奨
errorProcess("catchでエラーハンドリング").catch(error => {
    console.log(error.message);
});
```

resolveとrejectの使い方がわかったところで重要な以下の２点を覚えておいて次に進みます。

- Promise内のresolveメソッドが実行されるまで、then()の中身は実行されない
- Promise内のrejectメソッドが実行されるまで、catch()の中身は実行されない

### Promiseコンストラクタ内の例外処理

Promiseではコンストラクタの処理で例外が発生した`Promise`インスタンスは`reject`関数を呼び出したのと同じように処理されます。   
try...catch構文を使用しなくても自動的に例外がキャッチされます。

```JavaScript
function throwPromise() {
    return new Promise((resolve, reject) => {
        // Promiseコンストラクタの中で発生した例外は自動的にキャッチされreject関数が呼ばれる
        throw new Error("例外発生");
    });
}

throwPromise().catch(error => {
    console.log(error.message);
});
```

### Promiseの状態について
Promiseインスタンスには以下の3つの状態が存在します。

- Fulfilled  
resolve(成功)した時の状態。`onFulfilled`メソッドが呼ばれる

- Rejected  
reject(失敗)または例外が発生したときの状態。`onRejected`が呼ばれる。

- Pending  
FulfilledまたはRejectedではない状態。インスタンスを作成したときの初期状態。

これらは内部的な状態なのでこの状態を直接扱うことはできませんが、Promiseについて理解するのに役立ちます。

`Promise`インスタンスは作成時に`Pending`状態になり、処理の結果によって`Fulfilled`または`Rejected`に変化するとそれ以降変化しなくなります。

つまり、Promiseコンストラクタ内で一度resolveメソッドを呼び出すと、その後、rejectやもう一度resolveを呼び出したとしてもコールバック関数は1度しか呼び出されないことに注意する必要があります。

この１度きりのコールバック関数を登録するのが、`then`や`catch`といったメソッドです。


### 非同期処理の連結

ここまでPromiseオブジェクトについて説明してきました。
ここからはPromiseオブジェクトのありがたみをイメージできるケースを考えます。

単一の非同期処理の場合、Promiseオブジェクトを介する分、記述は冗長になります。
Promiseオブジェクトが真価を発揮するのは、複数の非同期処理を連結するような場合です。


```JavaScript
// 初回関数呼び出し
asyncProcess('初回')
.then(
    response => {
        console.log(response);
        // 初回の関数呼び出しに成功した場合、2回目を実行
        return asyncProcess('２回目');
    }
)
.then(
    response => {
        console.log(response);
    }
    error => {
        console.log(`エラー: ${error}`);
    }
);


```

この仕組みは、thenやcatchといったメソッドが新しい`Promise`オブジェクトを返すことで成り立っています。
これによって複数のthenメソッドをドット演算子で列記することができ、非同期処理を同期処理であるかのように書けます。(入れ子を深くせずに書けるという意味です)  

### 非同期処理の並列実行

非同期処理の直列実行の次は、並列実行のメソッドについて見ていきます。

- [Promise.all](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)メソッド  
Promise.allメソッドは複数の非同期処理を並列に実行し、そのすべてが成功した場合に処理を実行します。

```JavaScript
Promise.all([
    asyncProcess('1回目');
    asyncProcess('2回目');
    asyncProcess('3回目');
]).then(
    response => {
        console.log(response);
    },
    error => {
        console.log(`エラー: ${error}`);
    }
);
```

Promise.allでは、配列のかたちで渡された複数のPromiseオブジェクトがすべてresolveした場合にだけthenメソッドの成功時コールバック関数を実行します。
その際の引数(response)にはすべてのPromiseから渡された結果値が配列として渡されます。

Promiseオブジェクトのいずれかがreject(失敗)した場合には失敗コールバックが呼び出されます。

- [Promise.race](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)メソッド  
Promise.raceメソッドでは並列して実行した非同期処理のいずれか1つが最初に完了したところで成功時コールバック関数を実行します。
例えば、複数のデータベースレプリケーションに対して一斉にクエリを投げて最初に応答があったものを使用する、といった使い方ができます。
関数の命名通り、レースです。



## 参考

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function

https://jsprimer.net/basic/async/

https://www.amazon.co.jp/%E6%94%B9%E8%A8%82%E6%96%B0%E7%89%88JavaScript%E6%9C%AC%E6%A0%BC%E5%85%A5%E9%96%80-%E3%83%A2%E3%83%80%E3%83%B3%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%81%AB%E3%82%88%E3%82%8B%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E7%8F%BE%E5%A0%B4%E3%81%A7%E3%81%AE%E5%BF%9C%E7%94%A8%E3%81%BE%E3%81%A7-%E5%B1%B1%E7%94%B0-%E7%A5%A5%E5%AF%9B/dp/477418411X

https://qiita.com/ryosuketter/items/dd467f827c1b93a74d76


## まとめ

- 非同期処理にはもともとコールバック関数が使用されていたが、ネストの深さの問題や書きやすさから、ES2015以降はPromiseオブジェクトを使用する方法が主流。
- Promiseには内部的に3つの状態があり、1度状態変化した後は変化しない。
- Promiseのthenメソッドやcatchメソッドは新しいPromiseオブジェクトを返すため、thenやcatchメソッドをドット演算子でチェーンすることができ、例外処理がシンプルに書ける。

## 所感

今回はJavaScriptにおける重要な仕組みである非同期処理について調べてみました。  
個人的にJavaScriptはなにか作りたいときに都度調べるという方法で勉強してきましたが、調べて直感的に理解できなかったのが今回調査した非同期処理でした。  
知らない技術を調べる際にはその技術が解決する問題を先に知っておきどんな**つらみ**を解消するのかを意識することでより深い理解につながると思っています。  
また今回学んだPromiseオブジェクトを利用して個人開発のほうでなにか作ってみたいと思います。
