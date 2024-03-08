---
title: 駆け出しエンジニアから脱出するための道のり
tags: [onboarding]
createDate: 2024-03-07
updateDate: 2024-03-07
slug: engineer-onboarding-for-junior
---

## はじめに

今回は、エンジニアなりたての状態から何を学べばジュニアレベル、ミドルレベルのエンジニアとして活躍できるのかについて考えてみたいと思います。  
対象はWEB領域のエンジニアです。

## 前提

あくまで前提として、私の知っている範囲のことしか書けないので、弊社スマレジのエンジニアとして活躍できるレベルが目標です。  
スマレジでは、バックエンド、フロントエンドの領域で担当が分かれておらず、どちらも担当します。  
インフラについては専属のチームがありますが、プロダクトを担当しているエンジニアと相談しながらインフラ設計が行われることが多く、インフラについての知識がなくてもいいわけではありません。  

## ロードマップ

まずはロードマップとしてよく知られる図を確認してみます。
私自身が何を勉強すればいいのかわからなくなった時に参考にしていました。最近はあまり迷うことはないんですが、どうしてもエンジニアになりたてのときはわからないことが多すぎて、どれから勉強をすればいいのかわからず、わからないことがわかっていない状態に陥ることがありました。

- [Backend Developer Roadmap: What is Backend Development?](https://roadmap.sh/backend)  
- [Frontend Developer Roadmap: What is Frontend Development?](https://roadmap.sh/frontend)

## Leval 1: 全エンジニア必須スキル

ロードマップの両方に存在する要素

- Internet
  - How does the internet work? インターネットの仕組み
  - What is HTTP? HTTPとは何か?
  - Browses and how they work ブラウザとその仕組み
  - DNS and how it works DNSとその仕組み
  - What is Domain Name? ドメイン名とは？
  - What is hosting? ホスティングとは？
- Git
  - GitHub フレームワークやライブラリの調査など、最低限のGitHubに関する知識が必要
  - GitLab スマレジではGitLabを利用している

## Level 2: エンジニア基礎スキル

このレベルくらいまでは、個人の趣味レベルでも十分に習得できるものが多いと思います。  
それぞれの領域で本当に最低限の共通的な部分をLevel 2としてみました。

### ロードマップの両方に存在はするが、温度感が違うもの

- JavaScript
  - フロントエンドでは必須
  - バックエンドでは任意

### バックエンドの必須スキル

- バックエンド言語の習得
  - PHP スマレジの場合はPHPがメインとして採用されている言語
  - Java
  - C#
  - Javascript
  - Python
  - Ruby

- RDB
  - MySQL スマレジの場合はMySQLがメインとして採用されているRDB
  - PostgreSQL 一般的に採用されることの多いRDB
  - MariaDB
  - Oracle
  - MS SQL

### フロントエンドの必須スキル

- HTML
- CSS
- Package Manager
  - npm
  - yarn
  - pnpm
- Pick a Framework
  - React
  - Vue
  - Angular 選択肢としてはあるが、採用される機会がやや少ないイメージ

## Level 3: エンジニア実務スキル

ここからは実務で必要になるスキルを挙げてみます。  
これらの技術を実務に入る前に全て習得している必要はないと思いますが、都度必要になるタイミングでキャッチアップが必要です。

### 領域に関わらず学ぶべきスキル

- Web Security Basics
  - HTTPS
  - CORS
  - Content Security Policy
  - OWASP Security Risks

### バックエンド

- Learn about APIs
  - REST
  - JSON APIs
- Caching
  - CDN
  - Redis
- Testing
  - Unit Testing
  - Integration Testing
  - Functional Testing
- CI/CD
- Scaling Database
  - Database Indexes

----------------------------------------

~ 個人的に感じるジュニアとミドルの壁 ~

----------------------------------------

- More about Databases
  - ORMs
  - ACID
  - Transactions
  - N+1 Problem
  - Normalization
  - Failure Modes
  - Profiling Performance
- Software Design & Architecture
  - GOF Design Patterns
  - Domain Driven Design
- Design and Development Principles
  - Test Driven Development
  - CQRS
  - Event Sourcing

### フロントエンド

- TypeScript
- Writing CSS
  - Tailwind
  - CSS in JS ロードマップの記載はない
  - CSS Modules ロードマップの記載はない
- Build Tools
  - Module Bundlers
    - Webpack
    - Vite
    - esbuild
    - Rollup
    - Parcel
- Linters and Formatters ロードマップ上の優先度は低いが個人的には重要
  - ESLint
  - Prettier
- Testing your Apps
  - Vitest
  - Jest
- Server Side Rendering
  - Next.js
  - Nuxt.js
- Static Site Generators
  - Astro
  - Next.js
  - Nuxt.js

## スマレジのエンジニアに要求されるスキル

ここからは弊社スマレジの採用要件を見ながら、どのようなスキルが求められているのかを確認してみます。

### [WEBエンジニア](https://corp.smaregi.jp/recruit/job/web-engineer.php)

必要要件

- Webアプリケーション開発の実務経験（2年以上）
- バージョン管理、とくにGitを使った開発の経験

現在はPHPを利用していますが、PHPの経験は必須ではなく他言語においてWebアプリケーション開発の経験をお持ちの方であれば問題ありません。

歓迎要件

- リーダー経験
- Laravel、CakePHP経験
- Webアプリやスマホアプリでのパフォーマンスチューニング経験
- UnixライクOS環境下でのWebアプリケーションの開発経験
- AWSサービスの利用経験
- スクラムでのチーム開発の経験
- フロントエンド（Vue.js、またはReactを利用したUI開発経験、SPA開発経験）
- WebAPIを使用した開発経験、またはWebAPIの設計・実装経験
- Webアプリケーション開発に関係するインフラ知識・理解
- DDDやアーキテクチャ設計の知識・設計・実装経験

### [リードエンジニア/テックリード](https://corp.smaregi.jp/recruit/job/lead-engineer.php)

必要要件

- Webアプリケーション開発経験（PHP、Java、Rubyなど）
- 自社サービス/受託開発の開発・運用経験
- システムの課題解決に対する推進力
- 新技術における高速なキャッチアップ力

歓迎要件

- 業務システムの開発・運用経験
- 最新技術の習得および活用
- テックリード/リードエンジニアとしてチームの技術判断をした経験
- マイクロサービスのシステム運用経験

### 実際に必要だと感じるスキル


## 個人的につまづいたポイントとその解決方法

## 参考

https://qiita.com/mamimami0709/items/c9657367a8e7dfdca070
https://qiita.com/mamimami0709/items/fd6556707e4b924c65ab