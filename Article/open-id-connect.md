---
title: OpenID Connectについて
tags: [OIDC]
createDate: 2022-09-07
updateDate: 2022-09-07
slug: open-id-connect
---


### はじめに
今回はOpenID Connectについてです。   
認証の仕組みについて理解に乏しい部分があったため、調査したいと思います。   
今回はOpenID Connectを利用した認証の流れについて理解することを目的としています。   
暗号化やデータの形式については今回理解したいスコープとは異なるため、またの機会に調査しようと思います。   

## OpenID Connectとは
認証の仕組みの一種。   
本来、クライアント側で行っていた認証処理をOpenID Providerと呼ばれる他のサーバーに任せ、認証結果のみをクライアントが受け取って認証する仕組み。

## 認証と認可について
認証の仕組みであるOpenID Connectについて見るまえに認証と認可の違いについて調査しておく。
### 認証
英語では「Authentication」   
IT分野における文脈では、**通信の相手が誰であるかを確認すること**。   
例: マイナンバーカードを使用した本人確認   
純粋な認証というのは、誰かを確かめることであって、なにかを許可することではない。

認証における3つの要素

- WHAT YOU ARE   
相手にさまざまな情報(顔、声、指紋など、その人自身)を提示してアイデンティティ(その人であること)を確認させる方法

- WHAT YOU HAVE   
身分証や携帯電話などの本人しか持っていないものを提示することでアイデンティティの確認をする方法。   
身分証の場合は顔写真等がプリントされている場合もあり、その場合はWHAT YOU AREの要素も含んでいる。

- WHAT YOU KNOW   
パスワードや秘密の質問など、本人しか知り得ない情報を提示することでアイデンティティの確認をする方法。   
IT分野では最も多く使用される要素。

一般的には上記いずれかを満たすことで認証しますが、より確実な認証のためにワンタイムパスワードなどに挙げられるMFA(Multi-Factor Authentication)という考え方が採用されることもある。   
要するに認証とは、**本人であることをあの手この手で確かめること**。

### 認可
英語では「Authorization」   
IT分野における文脈では、**とある特定の条件に対して、リソースアクセスの権限を与えること**。   
例: 鍵の発行、チケットの発行   

純粋な認可は、認可されたからといって身元の証明とはなんの関係も無い。   
チケットを例に挙げると、チケットを購入した場合、電車に乗ることを許可される。
チケットを知人から譲り受けた場合でも、電車に乗ることを許可される。

認可は、認可情報を持っているという事実によって、なにか(リソースへのアクセス)が許可される。   
鍵の例で言えば、鍵を持っていればドアを開けることができる。ドアを開ける人が誰であるかはこのことには無関係。

### 認証と認可の分離
ここでややこしいのが、「多くの場合、認可は認証に依存している」ということ。

- 認可のみの場合(認証はしない)   
このパターンは考え得る。先程のドアやチケットの例のように特定のなにかを持っていればリソースへのアクセスができるパターン。

- 認証のみのパターン(認可はしない)   
このパターンは考えにくい。本人であることを検証してなにもしないパターンは無駄になるため、なんのために認証するのかわからない。   
ただ、今回のテーマであるOpenID Connectのように認証処理の委譲が発生するような場合、認証するシステムと認可するシステムが分離することはあり得る。

### 認証に基づく認可
一番に思いつく認証と認可のパターン。   
例: 運転免許証   
写真により誰かを認証した上で車の運転を許可している。他人に渡しても受け取った人が運転を許可されるわけではないのが先程のチケットの例とは違う。

### 認可に基づく認証
家の鍵を持っているから、家の持ち主本人である。と認証する場合。   
「OAuth認証」はこの仕組み。   
認証するために認可するというのは不要なリソースへのアクセスを許可しなくてはならない問題や、認可されているからといってその人本人とは言い切れない可能性がある。

認証と認可はよく比べられるが、

- 認証は証明書の検証(発行された証明書が正しいかどうか検証する)
- 認可は証明書の発行(証明書を発行し、持っている人にたいして何かを許可する)

なのでタイミングがそもそも違う。


参考: https://dev.classmethod.jp/articles/authentication-and-authorization/

## 従来の認証の流れ
ここから、今回のテーマであるOpenID Connectについて調査する。
以降、開発者が作成するシステムを**サービス**、OpenIDを発行するベンダを**OpenID Provider**とする。

OpenID Connectを使用しないサービスの従来の認証の流れは以下のようになる。

1. ユーザーは利用するサービスにIDとパスワードを使用してログイン
2. サービスはID・パスワード認証処理を行いデータベースに登録されているユーザー情報と一致すれば認証OKとする

ここで、ユーザーからの新規要望によりワンタイムパスワード処理を追加することになった場合、従来の認証の流れではサービスの開発者はワンタイムパスワード処理を自前で実装する必要がある。

## OpenIDConnectを使用した認証の流れ
OpenID Connectでは、今までサービス側で行っていたユーザーの認証をOpenID Provider側で行う。   
OpenID ProviderはGoogleなどを筆頭に様々なベンダがある。

1. ユーザーはOpenID Provider側にID・パスワードを入力
2. ワンタイムパスワードが必要な場合でも、OpenID Provider側に実装されているので入力
3. 認証に必要な情報を入力すると、OpenID Providerから認証に成功したという証を受け取れる
4. サービス側はその証を受け取り、OpenID Providerに提示する
5. サービス側から提示し証が正しいものかつ有効期限内であった場合、OpenID Providerはサービス側に**IDトークン**を発行する。

ここで、**従来の認証の流れ**と**OpenID Connectを利用した際の流れ**を比較すると、

- ID・パスワード認証、ワンタイムパスワード認証などの認証に関わる処理
- 認証に使用するID、パスワードなどの認証情報

の２つがサービス側からOpenID Provider側に移動していることがわかる。
サービス側で自前で実装するのではなく、OpenID Providerに認証にかかわる情報を任せることでセキュリティリスクの低減と、実装する手間の削減ができる。   
要するにOpenID Connectは、OpenID Providerに認証にかかわるプロセスを任せるためのプロトコル。

## IDトークンについて

先程の例で挙げた認証の流れの最後に登場した**IDトークン**について理解する。   
OpenID Connectは認証のための仕組みだが、本質は**IDトークンを発行**することにある。

詳細は以下参考記事にて解説されています。   
https://qiita.com/TakahikoKawasaki/items/8f0e422c7edd2d220e06

### IDトークンとは
以下で構成される文字列
```
ヘッダー.ペイロード(本文).署名
```
1つ目のピリオドまでがヘッダー、2つ目のピリオドまでがペイロード、以降が署名を表している。

例
```
eyJraWQiOiIxZTlnZGs3IiwiYWxnIjoiUlMyNTYifQ.ewogImlz
cyI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4
Mjg5NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAi
bi0wUzZfV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEz
MTEyODA5NzAsCiAibmFtZSI6ICJKYW5lIERvZSIsCiAiZ2l2ZW5fbmFtZSI6
ICJKYW5lIiwKICJmYW1pbHlfbmFtZSI6ICJEb2UiLAogImdlbmRlciI6ICJm
ZW1hbGUiLAogImJpcnRoZGF0ZSI6ICIwMDAwLTEwLTMxIiwKICJlbWFpbCI6
ICJqYW5lZG9lQGV4YW1wbGUuY29tIiwKICJwaWN0dXJlIjogImh0dHA6Ly9l
eGFtcGxlLmNvbS9qYW5lZG9lL21lLmpwZyIKfQ.rHQjEmBqn9Jre0OLykYNn
spA10Qql2rvx4FsD00jwlB0Sym4NzpgvPKsDjn_wMkHxcp6CilPcoKrWHcip
R2iAjzLvDNAReF97zoJqq880ZD1bwY82JDauCXELVR9O6_B0w3K-E7yM2mac
AAgNCUwtik6SjoSUZRcf-O5lygIyLENx882p6MtmwaL1hd6qn5RZOQ0TLrOY
u0532g9Exxcm-ChymrB4xLykpDj3lUivJt63eEGGN6DH5K6o33TcxkIjNrCD
4XB1CKKumZvCedgHHF3IAK4dVEDSUoGlH9z4pP_eWYNXvqQOjGs-rDaQzUHl
6cQQWNiDpWOl_lxXjQEvQ
```

この形式は[RFC 7515](https://www.rfc-editor.org/rfc/rfc7515)で定義されている形式らしい。   
JWS(JSON Web Signature)と呼ばれる形式。
**base64url**という方式でエンコードされたデータ。

### JWE(JSON Web Encryption)
また、以下のような**JWE**という形式のIDトークンも存在する([RFC 7516](https://www.rfc-editor.org/rfc/rfc7516))
```
ヘッダー.キー.初期ベクター.暗号文.認証タグ
```

この形式はIDトークンを暗号化したいときに使用される。
IDトークンの場合、4番目のフィールド、暗号文には`ヘッダー.ペイロード.署名`の形の平文を暗号化したものが入る。
つまりJWEの中にJWSが入る形式となる。

### JWT(JSON Web Token)
さらにJWT(JSON Web Token)、ジョットと呼ばれる形式がある。([RFC 7519](https://www.rfc-editor.org/rfc/rfc7519))
>「JWTとは、JSON形式で表現されたクレームの集合を、JWSもしくはJWEに埋め込んだもの」

IDトークンはJWTの一種で、クレーム、つまり認証に関わる情報を内包した平文を暗号化したもの。

## 振り返り
IDトークンについてざっくり理解したところで改めてOpenID Connectの具体的な流れについて振り返っておくと、以下のような流れになる。
先程の例より具体的な用語を使用している。

1. サービス管理者は、OpenID ProviderにクライアントIDとクライアントシークレットを発行してもらう。   
このクライアントIDとは、OpenID Connectを使用して認証に関する情報をOpenID Providerに委譲する際に、OpenID Providerからサービスに対して発行される固有のID。   
クライアントシークレットはクライアントIDに紐づくパスワード。

2. ユーザーはサービスを利用する際にOpenID Providerの認証画面へとリダイレクトされる(このとき、クライアントIDと認可コード等のクエリパラメータが渡される)。ユーザーは認証画面でID、パスワード、ワンタイムパスワードなどの認証情報を入力する。

3. 認証に成功するとOpenID Provider側で発行された認可コードとともにサービス側へリダイレクトされる。このときOpenID Providerは発行した認可コードを有効期限とともに保存している。

4. 認可コードを受け取ったサービスは、IDトークンをOpenID Provider側にリクエストする。

5. OpenID Providerは渡された認可コードが存在し、かつ有効期限内であれば**IDトークン**を返す。この形式が**JWT**と呼ばれる形式。

6. IDトークンを検証し、問題なければ認証OK


## 参考
[OpenID Connect Core 1.0 incorporating errata set 1](https://openid.net/specs/openid-connect-core-1_0.html)   
[RFC7515 JSON Web Signature(JWS)](https://www.rfc-editor.org/rfc/rfc7515)   
[多分わかりやすいOpenID Connect](https://tech-lab.sios.jp/archives/8651)   
[一番分かりやすい OpenID Connect の説明](https://qiita.com/TakahikoKawasaki/items/498ca08bbfcc341691fe)   
[OpenID Connect 全フロー解説](https://qiita.com/TakahikoKawasaki/items/4ee9b55db9f7ef352b47)   
[よく分かる認証と認可](https://dev.classmethod.jp/articles/authentication-and-authorization/)

## まとめ
今回はOpenID Connectについて調査しました。   
最近のサービスでは利用していないほうが珍しい仕組みだと思うのでその内容を詳しく勉強する機会になりました。   
認証と認可の違いについてもきちんと説明できるレベルまで自分の中に落とし込めた気がします。   
認証処理の委譲を行うことでセキュリティリスクの低減や実装コストの低減ができます。   
ただ、仕組み上OpenID ProviderへのIDとパスワードが漏洩してしまうと利用しているサービスすべてへ影響が出る点はデメリットかなとも感じましたが、それを超えるメリットがあると思います。   


