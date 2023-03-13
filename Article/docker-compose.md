---
title: Docker Composeについて
tags: [Docker]
createDate: 2023-02-11
updateDate: 2023-02-11
slug: docker-compose
---

## はじめに

前回の記事でDockerについて調査した続きです。  
今回は実際にアプリを作成するとなると使うことになるであろう`Docker Compose`について調査します。  
前回同様公式ドキュメントのGet startedの内容を確認し、自分なりの設定方法を考えてみます。  


## Docker Composeとは

複数コンテナのアプリケーションを定義、共有するためのツールです。  
YAML形式のファイルを作成することでコマンド1つで複数のコンテナを立ち上げたり、解体することができます。  

Macの場合、Docker Desktopをインストール済みであれば、Docker Composeのインストールは不要です。  
下記コマンドでバージョンが確認できればOKです。  
```
$ docker-compose version
```

## Composeファイルの作成方法

アプリのプロジェクトルートで`docker-compose.yml`という名前でファイルを作成します。

今回記載するコンテナの内容は下記になります。
```
$ docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

```yml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

```

- `version` ... docker composeのスキーマバージョン。詳細は[こちら](https://matsuand.github.io/docs.docker.jp.onthefly/compose/compose-file/)
- `services` ... コンテナの一覧を定義します。
- `app` ... コンテナ名です。任意の値に変更可能で、自動的にネットワークエイリアスになります。
- `command` ... コマンドを記載します。通常は`image`定義のすぐ近くに書きます。
- `ports` ... ポートを指定します。[記載方法](https://matsuand.github.io/docs.docker.jp.onthefly/compose/compose-file/compose-file-v3/#ports)がいくつかあります。今回の書き方は`HOST:CONTAINER`の書き方です。
- `working_dir` ... ワーキングディレクトリです。コマンドでいうと`-w`で指定したディレクトリです。
- `volumes` ... ボリュームの指定です。コマンドでいうと`-v` で指定したディレクトリです。ポート同様に[記載方法](https://matsuand.github.io/docs.docker.jp.onthefly/compose/compose-file/compose-file-v3/#volumes)がいくつかあります。Docker Composeにボリュームを定義する場合はカレントディレクトリからの相対パスで記載することができます。
- `environment` ... 環境変数の指定です。コマンドでいうと`-e`で指定していた部分です。

以上でアプリ用のコンテナの定義を記載できました。

続いてMySQLサーバーの定義を記載していきます。
```
$ docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

先程のアプリ用コンテナの定義の下にMySQL用のサービスを定義します。

```yml
 version: "3.7"

 services:
   app:
     # The app service definition
   mysql:
     image: mysql:5.7
```

次はボリュームマッピングの定義ですが、`docker run`を使用すると名前付きボリュームが自動生成されていました。  
Composeの場合は最上位項目として`volumes:`というセクションを作成し、サービス定義の中のマウントポイントをここに指定します。
ボリューム名だけを指定すれば、デフォルトのオプションが適用されます。[composeにおけるボリュームについてはこちら](https://matsuand.github.io/docs.docker.jp.onthefly/compose/compose-file/compose-file-v3/#volume-configuration-reference)


概要としては、docker compose自体が複数のコンテナの利用を前提としているため、マルチサービスにまたがって使用できる名前付きボリュームを生成します。

ボリューム自体の理解については[こちら](https://matsuand.github.io/docs.docker.jp.onthefly/storage/volumes/)

ボリュームが設定できれば、後は必要な環境変数を`environment`として定義します。

ここまでで完成した`docker-compose.yml`が以下のようになります。

```yml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```


アプリケーションの起動には、`docker-compose up -d`コマンドを利用します。  
`-d`オプションの指定でバックグラウンドで実行されるようになります。  

ちなみに、Docker Composeを利用するとネットワークは自動的に生成されます。  

ログを確認する場合は`docker-compose logs -f`コマンドを実行することでサービスのログを1つにまとめて表示する事ができます。  
`-f`コマンドを指定するとログ出力を継続する事ができます。また、特定のサービスのログのみを確認したい場合は`docker-compose logs -f app`のようにログコマンドの最後にサービス名を指定します。


アプリケーションのコンテナをまとめて削除する場合は`docker-compose down`コマンドを実行します。  
この場合は名前付きボリュームは`docker-compose down`では削除されないため、`--volumes`フラグをつける必要があります。  

### ここまでのまとめ

ここまでの内容で公式ドキュメントにあるチュートリアルについて確認しました。

- docker composeを利用することで複数コンテナの立ち上げ、削除をコマンド一つで簡単に行う事ができる。
- docker composeファイル作成の際には[ボリューム](https://matsuand.github.io/docs.docker.jp.onthefly/storage/volumes/)、[ネットワーク](https://matsuand.github.io/docs.docker.jp.onthefly/network/)の理解があると楽


## ベストプラクティス

こちらに[イメージビルドのベストプラクティス](https://matsuand.github.io/docs.docker.jp.onthefly/get-started/09_image_best/)がまとまっています。  

この中で、ぱっと理解できなかったキャッシュ処理とマルチステージビルドについてまとめます。

## レイヤーのキャッシュ処理

Dockerは`1つのレイヤーに変更が入ると、それ移行に続く全レイヤーは再生成されます。`  

イメージがどのように構成されているかは`docker image history`コマンドで確認できます。  

Dockerfileの各コマンドはイメージ内の1つのレイヤーに対応しています。  

つまりイメージに変更があった場合、`yarn install --production`が再度実行され、依存パッケージが再インストールされます。  
これをビルドのたびに何度もインストールするのは無駄なため、キャッシュ処理を考えます。  

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

Nodeベースのアプリケーションの場合、依存パッケージは`package.json`ファイルに定義されます。  
このファイルに変更があった場合にのみyarnによる依存パッケージの更新を行うにはどうすればいいでしょうか。  

```dockerfile
 # syntax=docker/dockerfile:1
 FROM node:12-alpine
 WORKDIR /app
 COPY package.json yarn.lock ./
 RUN yarn install --production
 COPY . .
 CMD ["node", "src/index.js"]
```

Dockerfileと同じフォルダー内に`.dockerignore`という名前のファイルを生成して、その内容を以下とします。
```
node_module
```
[.dockerignore](https://matsuand.github.io/docs.docker.jp.onthefly/engine/reference/builder/#dockerignore-file)ファイルを利用することでDockerCLIはそこに記述されたパターンにマッチするようなファイルやディレクトリを除外してコンテキストを扱います。  
今回、`node_modules`フォルダを記載しておくことで`RUN`コマンドによって生成されたファイルを上書きしてしまうことを避ける事ができます。  
node.jsを利用したベストプラクティスは[こちら](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)

下記コマンドを実行して新たなイメージをビルドするとキャッシュが使用されていることを確認できます。  
キャッシュが使用されている箇所では`Using cache`と出力されます。
```
$  docker build -t getting-started .
```
## マルチステージビルド

[マルチステージビルド](https://matsuand.github.io/docs.docker.jp.onthefly/develop/develop-images/multistage-build/)とは、イメージの生成に複数ステージを利用するというツールです。  

- ビルド時の依存パッケージと実行時の依存パッケージを分離します。
- アプリとして実行する必要のあるものだけを作り出すことによって、イメージ全体のサイズを削減します。

マルチステージビルドを行うには、Dockerfile内に`FROM`行を複数記述します。  
各`FROM`命令のベースイメージはそれぞれ異なるもので、各命名から新しいビルドステージが開始されます。  
これを利用して片方のビルドステージで生成した内容を他方にコピーして破棄するといった使い方ができます。  

例えば公式にあるReactアプリケーションの例を見てみます。  

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

上記の`node:12`イメージはビルド処理を行ったあと、その出力結果をnginxコンテナにコピーします。  
ビルドが終了したあとのコンテナは放置されます。  


`FROM node:12 AS build`のように指定していますが、これはビルドステージを表します。  
デフォルトではビルドステージに名前はつかず、最初の`FROM`命令を0として順番に整数値が割り振られます。  
`FROM`命令に`AS <NAME>`という構文を加えることでステージ似名前をつける事ができ、`COPY`命令においてその名前を使用しています。  
`COPY --from=build /app/build /usr/share/nginx/html`の部分です。  

また、`--target`を指定するとイメージをビルドする際に特定のステージのみを対象とすることもできます。  
この機能を使用すると、デバッグ用に`debug`ステージを用意してデバッグツールを導入し、本番の`production`ステージではスリムなイメージを使用する事ができます。また、テストの場合も同様です。  

このように、イメージがどう構成されているかを理解できれば、イメージのビルドをより効率的にすることができます。  
キャッシュを利用することでビルドがより早くなり、マルチステージビルドをうまく使えばイメージサイズ全体を小さくする事ができます。  

## まとめ

- docker composeを利用することで複数コンテナの立ち上げ、削除をコマンド一つで簡単に行う事ができる。
- docker composeファイル作成の際には[ボリューム](https://matsuand.github.io/docs.docker.jp.onthefly/storage/volumes/)、[ネットワーク](https://matsuand.github.io/docs.docker.jp.onthefly/network/)の理解があると楽
- マルチステージビルドを利用して基本的にイメージのサイズを小さくするように工夫ができる
- キャッシュを利用してビルドは必要なときに必要なだけ行うようにする

## 所感

あらためて公式ドキュメントを読むとまだまだDockerを雰囲気で使ってしまっていたなと反省しました。  
今回である程度基本は理解できたので、CI/CDでの活用やクラウドへのデプロイ時にコンテナを利用してみたいと思います。  
これらはまた個人開発等で使用してみる際に再度調査してみたいと思います。  