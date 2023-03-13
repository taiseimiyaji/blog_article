---
title: TypeScriptでAtCoderに挑戦してみる
tags: [TypeScript]
createDate: 2023-01-17
updateDate: 2023-01-17
slug: typescript-atcoder
---

## はじめに

スマレジでは、副業が可能です。   
https://corp.smaregi.jp/recruit/culture/workstyle.php   

個人的に最近、副業にも挑戦しようかなと考え始めています。   
副業に割ける稼働時間的には週10h程度となりそうなので、比較的タスクを細かくしやすいフロントエンドの案件を中心に探していたりします。   
担当するプロダクト的にはバックエンドを中心に開発してきましたが、最近TypeScriptやReactに興味があるので、そちらのキャッチアップの加速にも役立つかなと考えています。   
そこで今回はTypeScriptに慣れることと、コーディングテスト等への対策としてAtCoderの問題をTypeScriptで解いてみたいと思います。

## AtCoder

競技プログラミングと呼ばれるプログラミングコンテストサイトです。  
後の説明は省略します。

https://atcoder.jp/?lang=ja

過去問は下記サイトから閲覧できます。

https://kenkoooo.com/atcoder/#/table/

なお、問題に色がついていますが、
```
灰色 -> 茶色 -> 緑色 -> 水色 -> 青色 ...
```

の順番で難しくなっていきます。   
この辺りのレートのシステムについても公式を参考にしていただければと思います。   

以前少しやったことがあるんですが、その際の**個人的な感触**としては以下の様な感じです。   
業務上はC問題くらいはすらすら解けるといいな〜と思ってたりします。   

```
A問題...標準入出力とプログラミングの基礎がわかれば誰でも解ける   
B問題...問題文をきちんと理解できてfor文とif文が書ければ愚直に解ける   
C問題...愚直に解くだけでは大抵計算量で弾かれてしまうのでなにかしら対策が必要   
D問題以降...何回か挑戦したけど試行回数少なくてあまりわかってない   
```

## 環境構築

前提として、真面目に競技プログラミングに取り組む場合はTypeScriptはおすすめしません。  
大人しくC++かPythonを使用するべきだと思っています。  
が、今回はTypeScriptに慣れることが目的のため、あえてTypeScriptを使用していきます。  

nodeやnpmのインストールは今回は省略します。  
まずはパッケージ管理のためにpackage.jsonを下記コマンドで作成します。  
```shell
npm install
```

必要なものは以下のパッケージとなります。

- typescript@3.8 (本記事執筆時点でのAtCoderでのTypeScript対応バージョン)
- ts-node (TypeScriptのまま実行するためのライブラリ)
- @types/node (TypeScriptの標準入出力ライブラリ等を含むライブラリ)

忘れずに`tsconfig.json`の作成もしておきましょう。
```shell
npx tsc --init
```

## 標準入出力
TypeScriptで一番めんどくさいのが標準入出力になります。

windowsでは使えないですが、以下が一番シンプルな記述かと思います。
```TypeScript
const input = require('fs').readFileSync('/dev/stdin', 'utf8');
```

毎回書くのも面倒なのである程度テンプレートとしておくのがいいと思います。  
Ctrl + Dで入力終了です。  
他にもいくつか手段はあるので気になる方がいれば調べてみてください。

## 簡単な問題に挑戦!(ABC285)

以下、AtCoderの問題にいくつか挑戦してみます。   
正解できるコードではあるはずですが、より効率的なものやわかりやすい処理等あると思いますのであくまで参考としてください。   

### A問題 Edge Checker 2
まずは一番簡単なA問題に挑戦してみます。   
記事執筆時の直近のA問題に挑戦します。

https://atcoder.jp/contests/abc285/tasks/abc285_a

- 考え方

競技プログラミングではある程度スピードが求められるので、問題を見て実装方法を判断します。   
今回のA問題では2パターンぱっと思いついたのでそれぞれ考え方を整理しておきます。   

1. 規則性を見つける方法   
直接結ばれている親と子には規則性があります。   
親の数字をnとしたとき、`2n`,`2n+1`であれば直接結ばれています。   

上記を考慮してコードを書くと以下の様になります。(競技プログラミングの特に簡単な問題では変数名の命名が適当になりがちなんですがご容赦ください)
```TypeScript
import * as fs from 'fs';

const input = fs.readFileSync("/dev/stdin", "utf8").split(/\s/);
const a = +input[0];
const b = +input[1];

const ans = (Boolean)(b === (2 * a) || b === (2 * a + 1));

if (ans) {
    console.log("Yes");
} else {
    console.log("No");
}
```

2. 愚直に書く方法   
これはA問題のようなパターンが少なく、制約がある問題で使える方法なんですが、愚直に全てのパターンを実行することでも正解することができます。
説明するよりコードを見てもらった方が早いと思います。

```TypeScript
import * as fs from 'fs';

const input = fs.readFileSync("/dev/stdin", "utf8").split(/\s/);
const a = +input[0];
const b = +input[1];

const tree =
{
    1: [2, 3],
    2: [4, 5],
    3: [6, 7],
    4: [8, 9],
    5: [10, 11],
    6: [12, 13],
    7: [14, 15],
};
console.log(tree[a] && tree[a].includes(b) ? "Yes" : "No");
```

`console.log()`の中で`tree[a]`の存在をチェックして7より大きい場合は全てNoとしています。  
あとは愚直にオブジェクトを使用しています。   
今回の様なわかりやすい例ではあまり使用しませんが、いざコンテストとなると早く解かないとという焦りなどからA問題で混乱してしまうこともあるので、この様な愚直な解き方も覚えておくと使えるかもしれません。   

### B問題 Longest Uncommon Prefix

https://atcoder.jp/contests/abc285/tasks/abc285_b

個人的にB問題を解く時の注意点は仕様の誤解をしない様にすることです。  
愚直な方法で解けることが多いですが、少し問題文が複雑になっていたりするので、そこだけ気をつければ解ける問題が多いと思います。(もちろんたまに難しいやつもあります)  

いや難しい、、、。普通に問題の意味を理解するのに時間かかりました。  
しかもバグらせて一回WA(不正解)出しました、、、。

考えた方法を整理していきます。  

1. まずはTypeScriptの標準入力でちゃんと数値と文字列として受け取る(ここで少し調べたりしてロス)
2. 問題文を見てループ変数`i`のループをまず書く
3. 問題文のケースを見てチェックするためのループ(ループ変数が`k`)のものを考える。
4. i = 1の時チェックはS5まで行う(最後のチェックはS5とS6)
5. i = 2の時チェックはS4まで行うが途中で条件を満たしbreak
6. i = 3の時チェックはS3まで行う(最後のチェックはS3とS6)
7. ここまででkの最大値が`n-6`であることがわかる。また、チェックする文字は(SkとSk+i)
8. 求めるのは条件を満たす最大値`l`なので、問題の条件を満たしていれば更新、条件を満たさない場合はbreakする;
9. 問題文より、breakする条件は`Si` === `Sk+i`のとき。

上記をもとにまずは愚直にループを回してみます。

```TypeScript
import * as fs from 'fs';

const input = fs.readFileSync("/dev/stdin", "utf8").split(/\s/);
const n: number = parseInt(input[0]);
const s: String = input[1];

for (let i = 0; i < n; i++) {
    let l = 0;
    for (let k = 0; k < n - i; k++) {
        if (s.charAt(k) === s.charAt(k + i)) {
            break;
        } else {
            l++;
        }
    }
    console.log(l);
}
```
愚直に用件を実装しましたがこれで正解(AC)取れたのでよしとします。

### C問題  

C問題以降は多少実装に工夫が必要になる場合があります。  
今回の問題の鍵は問題文をみて入力文字列を26進数の数値だと考えることだと思います。  
と、個人的に考えていたのですが公式解説を回答後に確認すると26進数として考えるのが一番目の解説方法じゃなかったです。  
類題をみたことがあったのでパッと解法思いつきましたが大変なのはここからでした。  

- 一回目の提出(WA)

```TypeScript
import * as fs from 'fs';

const input = fs.readFileSync("/dev/stdin", "utf8").split(/\s/);
const s: String = input[0];

const length = s.length;
const charCodeAt = 64;
let num = 0;
for (let i = 0; i < length; i++) {
    num += (s.charCodeAt(i) - 64) * (26 ** (length - (i+1)));
}
console.log(num);
```


上記の提出で50ケース中47ケースACで3ケースWAとなりました。   
このパターンが一番きついですね。ある程度のロジックは合っていますし、人間は自分の書いたコードを信じたがる業の深い生き物なので。  

経験上ですが、こういったほぼほぼ正解でいくつかのテストケースのみ落ちる場合は境界値系のテストで落ちている可能性が高いです。   
今回の場合はJavaScriptのnumber型の最大値を超えていました。   
BigIntを使用するよう修正します。合わせて少し解説コメントもいれてみやすくしました。   

```TypeScript
import * as fs from 'fs';

const input = fs.readFileSync("/dev/stdin", "utf8").split(/\s/);
const s: String = input[0];

const length = s.length;
const charCodeAt = 64;
let num = BigInt(0);
for (let i = 0; i < length; i++) {
    const charNum = (s.charCodeAt(i) - charCodeAt); // アルファベットを文字コードを使用して数値に変換
    // 1桁目の場合は　26の0乗 * 文字列の番号
    // 2桁目の場合は　26の1乗 * 文字列の番号
    num += BigInt(charNum * (26 ** (length - (i+1))));
}
console.log(num.toString());
```


## 所感

今回はAtCoderにTypeScriptで挑戦してみました。  
C問題くらいまでであれば言語に関係なく解くことができそうです。(D問題以降は挑戦してみる価値はありそう)  
今回はB問題で思いのほか時間を取られましたが、C問題はさくっと解くことができました。  
これからもアルゴリズム力の強化のためにコンテストに参加してみたいと思います。  
TypeScriptは使用しているユーザーは多いと思うので興味があれば過去問等からでも解いてみてください。  