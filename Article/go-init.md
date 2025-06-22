---
title: 生成AI時代に備えてGo言語を学ぶ
tags: [go]
createDate: 2025-06-22
updateDate: 2025-06-22
slug: go-init
draft: false
---

## はじめに

生成AIと相性の良い言語として、これまで中心的にTypeScriptを利用していましたが、バックエンドを書くにあたって、Go言語を使いたい場面がでてきたので、Go言語を学び始めます。

生成AIと相性の良い部分として、

- 言語構造がシンプルであること
- 型があること
- Rustほどビルドに時間がかからないこと

などが挙げられます。また、Go言語を書くうえで辛い部分だったシンプルな仕様であるがゆえのコード量が多くなる問題を生成AIが解消してくれるのでは？と考えています。

## 環境構築

### Go言語のインストール

- Go言語の公式サイトから言語のインストーラをダウンロードしてインストールする方法

https://go.dev/

- Dockerを使ってGo言語の開発環境を構築する方法

インストールしてローカルの環境を汚すのがいやだったのでDockerを使って環境構築をしてみます。

```Dockerfile
FROM golang:1.24-alpine

WORKDIR /app

# Goモジュールの依存関係をキャッシュ
COPY go.mod go.sum* ./
RUN go mod download 2>/dev/null || true

COPY . .

# ホットリロード用のairをインストール
RUN go install github.com/air-verse/air@latest

CMD ["air"]
```

シンプルなDockerfileとcompose.yamlを用意します

```yaml
services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app
    ports:
      - "8080:8080"
    environment:
      - CGO_ENABLED=0
    tty: true
```

### ブラウザからの確認

自分がWEBエンジニアなので、技術的に理解しているものをGoでも作成してみます。

最終的にWEB APIサーバーを構築することを目指すので、ブラウザに実行結果を表示してみます。

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, Go with Docker Compose!")
    })

    fmt.Println("Server starting on port 8080...")
    if err := http.ListenAndServe(":8080", nil); err != nil {
        panic(err)
    }
}
```

以下の２つのパッケージを利用します。

- "fmt": 標準出力や文字列整形を行うためのパッケージ。
- "net/http": HTTPサーバーやクライアントを扱う機能を提供するパッケージ。

このコードの状態でコンテナを立ち上げるとブラウザ側で表示されます。

```bash
docker compose up
```

```go
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, Go with Docker Compose!")
    })
```

の部分ではHTTPリクエストのルーティング設定を行っています。

1. http.HandleFuncの部分でパス`/`に対するリクエストが来たときに呼び出す関数を登録しています。
2. 無名関数 func(w http.ResponseWriter, r *http.Request) が実際の処理を担当。

- w http.ResponseWriter: レスポンスを書き込むためのインターフェース。
- r *http.Request: リクエスト情報を格納した構造体へのポインタ。

fmt.Fprintf(w, "..."): レスポンスボディに文字列を書き込む。ここでは "Hello, Go with Docker Compose!" が返されます。

## Goにはクラスがない

ここまでは基本的な環境構築と簡単な関数を書きました。

ここで、WEBでバックエンドを構築する際によく利用される他の言語と比べた際の特徴として、オブジェクト指向のクラスがGo言語には存在しないことを知りました。

どういう思想でクラスが存在しないのか、ドメイン駆動等の場合にどうやって設計をするのかについて考えてみます。

まず、シンプルなUserエンティティをPHPで書くとこんな感じです。

```php
class User
{
    private function __construct(
        private UserId $id,
        private UserName $name,
        private Email $email,
    ) {
    }

    public function getId(): UserId
    {
        return $this->id;
    }

    // 残りのゲッターやセッター
}
```

このように、PHPではクラスを使ってオブジェクトを作成します。

Goにはクラスがないので、関連する情報をひとまとめにする構造体(struct)と、その構造体にメソッドを紐づけて利用してクラスっぽいことをするようです。

また、特徴的な部分として、ファイル単位ではなくパッケージ単位で可視性を制御する点も特徴です。

```go

// domain/user.go
package domain

// User はドメインのエンティティを表す構造体
type User struct {
    id    UserID
    name  UserName
    email Email
}

// NewUser は User のコンストラクタ相当の関数
func NewUser(id UserID, name UserName, email Email) *User {
    return &User{id: id, name: name, email: email}
}

// GetID はエクスポートされたメソッド（パッケージ外から呼べる）
func (u *User) GetID() UserID {
    return u.id
}

// その他のゲッターや振る舞いも同様に定義
```

### パッケージとは

Goのソースコードはパッケージで構成され、同じパッケージ内のファイル同士であれば相互にプライベートなフィールドやメソッドにアクセスできます。
ちなみに、Goではプライベートなフィールドやメソッドは小文字で始まる名前にします。

役割としては、

- 名前空間として型や関数の衝突を回避する
- ファイルをまたいで同一パッケージの要素を共有する
- 公開範囲をパッケージの単位で制御する

といったものがあります。

### 構造体とは

- クラスのように複数のフィールドをまとめて持つ事ができます
- クラスと違い、継承ができません。
- メソッドは構造体内部にあるわけではなく、構造体を利用する形で定義します。
- この構造体 + それに紐づくメソッド というかたちで他の言語でいうクラスに近い操作をします。

### クラスがないことによるメリット

Go言語に関するよくある質問は下記にまとめられています。

- [よくある質問（FAQ） - Goプログラミング言語](https://go.dev/doc/faq)

この中で何故型の継承がないのか、という問いがあり、要約すると次のようになります。

```markdown
Go では、事前に型同士の継承関係を宣言せず、型が定義するメソッドの集合がインターフェースの要件を満たしていれば自動的にそのインターフェースを実装したとみなします。この仕組みによって、

* 余計な宣言（ブックキーピング）が不要
* 伝統的な多重継承の複雑さがなくなる
* 単一メソッドの小さなインターフェース（たとえば `io.Writer`）でも有用な概念を表現できる
* 既存の型をあとから新しいインターフェースに適合させられる（テスト用などにも便利）
* 型とインターフェースの明示的な階層関係を管理する必要がない

たとえば、`fmt.Fprintf` はファイルだけでなく任意の出力先に書き込めますし、`bufio` パッケージはファイル I/O とは独立して動作します。Go のシンプルなインターフェース設計は、プログラムの構造にも大きな影響を与える生産的な仕組みです。
```

この仕組みはTypeScriptの構造的型(structural typing)と似ていますね。

ただ、Goの場合は型自体は名前的に定義されるので、同じメソッドを持っていても別物として扱い、インターフェースへの適合は構造的に判定されるようです。

言い換えれば、「この型がインターフェースの要求するメソッドをすべて持っていれば実装済みとみなす」方式ということですね。

## まとめ

今回はGo言語の基本的なデータ構造について学びました。

最近、RustやGoといった静的型付け言語に触れて生成AIとの相性を確認しているのですが、現時点ではビルド時間面ではGoを利用したい気持ちが強いです。

また、言語構造がシンプルな点も魅力的で、もともとC言語を触っていたのであまり違和感なくGo言語を使えるかもしれません。

引き続きWEB APIサーバーをGoで作成していきたいと思います。