---
title: OpenAPIの理解とその活用方法を考える
tags: [OpenAPI]
createDate: 2024-08-28
updateDate: 2023-08-28
slug: open-api
---

## はじめに

今回はWeb APIの仕様書として広く使われているOpenAPIについて調査してみます。

## OpenAPIとは

OpenAPI(旧Swagger)はREST APUのAPI記述形式です。

公式ドキュメントによると、次の内容を記述できると書かれています。

- 利用可能なエンドポイント(/users)と書くエンドポイントでの操作(GET /users, POST/users)
- 操作パラメータ 各操作の入力と出力
- 認証方法
- 連絡先情報、ライセンス、利用規約、その他の情報。

参考: https://swagger.io/docs/specification/about/

なお、よく登場するSwaggerという用語は、OpenAPIの前身の名称のようです。  
もともとReverb(現SmartBear)が開発したAPI設計ツールおよび仕様です。  
APの設計やドキュメント化、テスト、モックサーバーなどシミュレーションのためのツールセットを提供しており、広く使われるようになりました。  
OpenAPIはそのSwagger仕様をベースに進化したもので、Swagger 2.0の後継として誕生しました。現在は[OpenAPI Initiative](https://www.openapis.org/)という組織によって管理されているAPIの設計と仕様を標準化するための業界標準です。  

現在Swaggerという名称は、OpenAPIのためのツールの名称として残っており、Swagger EditorやSwagger UIなどがOpenAPIのツールとして提供されています。  
OpenAPIという名称は先述の通り、ツールではなく仕様の名称です。

## OpenAPIの活用方法、事例

基本的な活用としては、OpenAPIでREST APIの仕様を記述することですが、ほかにも次のような活用方法があります。

1. 自動テスト
いわゆる契約テストとして、OpenAPI仕様書をもとに、APIが仕様通りに動作しているかを自動的にテストするツールがあります。  
例えば、GUIでAPIリクエストを送信できるクライアントツールとしてPostmanがあります。  
このPostmanのリクエストを定義したCollectionをOpenAPI仕様書から自動生成するためのツールもあります。([OpenAPI 3.0, 3.1 and Swagger 2.0 to Postman Collection](https://github.com/postmanlabs/openapi-to-postman))  
PostmanはGUIなので、CI/CDパイプラインに組み込むことは難しいですが、Postmanのコレクションをコマンドラインで実行する[Newman](https://github.com/postmanlabs/newman)というツールもあるようです。

2. モックサーバーの作成
開発の初期段階で、バックエンドチームとフロントエンドチームが分かれている場合、OpenAPI仕様を先に作成することで、フロントエンドチームはモックサーバーを使って開発を進めることができます。
これにより、バックエンドの実装が完了する前に、APIとのやりとりを模倣しながら開発を進めることができます。ツールはPrismが一番有名なのかなと思います。
OpenAPI仕様書以外にも、PostmanのCollectionからモックサーバーを作成することもできます。[Postman Collections Support](https://docs.stoplight.io/docs/prism/bffdc5c112ca9-postman-collections-support)

3. クライアントコードの自動生成
OpenAPI仕様書から、各種プログラミング言語向けのクライアントSDKを自動生成できます。ツールは割と乱立していそうですが、有名どころは下記の通りです。

- Swagger Codegen
- OpenAPI Generator
- Kiota

一番ポピュラーなのはSwagger Codegenかなと思います。先に述べたSwaggerツール群の一つです。OpenAPI GenerattorはSwagger Codegenからのフォークで、Swagger Codegenの後継として開発されています。(https://github.com/OpenAPITools/openapi-generator/blob/e78aeb6bc7610a763e17e3d53614a1ef1990f311/docs/qna.md)

4. API Gatewayの設定
OpenAPI仕様から、AWS API GatewayのCDKを利用してAPI Gatewayの設定を自動生成することができるようです。参考: https://zenn.dev/taroman_zenn/articles/91879cec40627c

## スキーマ駆動開発

OpenAPIはREST APIの仕様を記述するためのもので、スキーマ駆動開発の一環として使われることが多いです。  
スキーマ駆動開発とは、実装前にAPIの仕様をスキーマとして定義し、そのスキーマに従って実装を進める開発手法です。  
これにより、先述したようなドキュメント、実装コード、テスト、モックサーバーなどの一部を自動生成することができます。  
また、バックエンドチームの実装を待たずにフロントエンドチームが開発を進めることができるようになります。  
また、スキーマを決めることによってデータ構造等の一貫性を保つことができるため、開発の品質を向上させることができます。

個人的に実現したいスキーマ駆動開発のポイントとして、次のようなものがあります。

- OpenAPI仕様書からTypeScriptの型定義を自動生成する。
- バックエンドでは実装と仕様の乖離を減らす。

一点目についてはさまざまなツールが存在しているので、実現は難しくないと思います。例えば、[swagger-typescript-api](https://github.com/acacode/swagger-typescript-api)のようなツールがあります。

2点目については、バックエンドをtypescript以外で実装する場合は同じ言語で型定義を共有することができないため、仕様と実装の乖離が生じやすいです。
PHP、Laravelの場合はこちらの記事が参考になりそうでした。  
参考: [実装と乖離させないスキーマ駆動開発フロー / OpenAPI Laravel編](https://zenn.dev/katzumi/articles/schema-driven-development-flow)

スキーマとコードを同期させるためには、コードベースに埋め込むのがいい、という発想は以前phpDocumentorを利用した経験があるので、なるほどと思いました。

PHP8以降はAttributeが利用できるようになったので、phpDocよりもAttributeの方がスキーマとコードを同期させやすいかもしれません。

[swagger-php](https://github.com/zircote/swagger-php)では、Attributeでスキーマを記載できます。  
IDEのサポートを受けることができる点もAttributeの利点です。

ただ、参考記事のようにAttributeから全てのコードを生成するのは、すでに動いているプロジェクトに導入するのは難しいかなと感じました。

あくまで補助的に導入し、IDE等で警告を出すなどの形でスキーマとコードの乖離を防ぐのが良いのかなと思いました。  

バックエンドの実装を待たなくてもいいことがスキーマ駆動開発のメリットでもあるため、各プロジェクトで、先にOpenAPI仕様書を作成するのか、コードベースから生成するのかを検討する必要がありそうです。

ドキュメントの重要性は叫ばれますが、実際にはドキュメントはコードベースと乖離してしまい、信頼できなくなった時点で無価値になってしまいます。  
そのため、スキーマ駆動開発はドキュメントの信頼性を高めるためにも有効な手法だと感じました。テスト駆動開発に近い感覚で開発を進めることができると思います。

実装の乖離のチェックとして、契約テストを行うことも有効だと思います。Pact、Prism、Postmanなどのツールを使って、APIの仕様と実装が一致しているかを自動的にチェックする仕組みの用意、CI/CDパイプラインでの実行が現実的に導入しやすいと感じました。

## まとめ

- OpenAPIはREST APIの仕様を記述するための仕様書です。
- OpenAPIを使うことで、自動テスト、モックサーバーの作成、クライアントコードの自動生成、API Gatewayの設定などができます。
- スキーマ駆動開発のポイントとして、OpenAPI仕様書からTypeScriptの型定義を自動生成することが挙げられます。
- バックエンドの実装を待たなくてもいいことがスキーマ駆動開発のメリットでもあるため、各プロジェクトで、先にOpenAPI仕様書を作成するのか、コードベースから生成するのかを検討する必要がありそうです。
- スキーマ駆動開発はドキュメントの信頼性を高めるためにも有効な手法だと感じました。テスト駆動開発に近い感覚で開発を進めることができると思います。
- Pact、Prism、Postmanなどのツールを使って、APIの仕様と実装が一致しているかを自動的にチェックするCI/CDパイプラインの実行が現実的に導入しやすいと感じました。
