# Codexからブログ公開までの作業手順

このドキュメントは、このPC上のCodexを使って記事HTMLを作成し、`taiseimiyaji/blog_article` にPRを作成し、GitHub Actionsで `taiseimiyaji/astro-blog` へ同期するための運用手順をまとめたものです。

## 全体像

```text
User
  -> Codex
  -> taiseimiyaji/blog_article
  -> blog_article PR
  -> Human Review
  -> blog_article main
  -> GitHub Actions
  -> taiseimiyaji/astro-blog sync PR
  -> astro-blog build
  -> Blog publish
```

`blog_article` は記事生成・記事素材管理の入口リポジトリです。Codexは基本的にこのリポジトリだけを操作します。

`astro-blog` は公開ブログ本体です。Astroのテーマ、レイアウト、スタイル、ビルド設定を持ち、`blog_article` から同期された記事を受け取ります。

## 記事配置ルール

Codexが生成する記事は、`Article/note/` 配下に追加します。

```text
Article/
  note/
    2026/
      06/
        2026-06-14-codex-blog-workflow.html
```

命名ルールは以下です。

- ディレクトリ: `Article/note/YYYY/MM/`
- ファイル名: `YYYY-MM-DD-slug.html`
- slug: 英小文字、数字、ハイフンのみ
- 画像を使う場合: `images/YYYY/MM/` 配下に置く

既存記事は `Article/` 直下にもありますが、Codex生成記事は新規運用として `Article/note/` に集約します。

## メタデータルール

`astro-blog` 側で扱いやすいように、HTMLファイルの先頭には次のメタデータコメントを付けます。

```html
<!--
title: "Codexで質問からブログ記事PRを作る運用"
tags: ["AI", "Codex", "Blog"]
createDate: 2026-06-14
updateDate: 2026-06-14
slug: 2026-06-14-codex-blog-workflow
draft: false
format: html
-->
```

必須項目です。

- `title`
- `tags`
- `createDate`
- `updateDate`
- `slug`
- `draft: false`
- `format: html`

この運用では、記事追加時点で `draft: false` にします。公開可否はPRレビューで判断し、`main` へのmergeを公開承認として扱います。

## 記事化する入力

Codexに記事作成を依頼するときは、入力の先頭に `/blog` を付けます。

```text
/blog
以下の内容を元に、ブログ記事を作成してください。

テーマ:
Codexで質問からブログ記事を作成し、GitHub上にPRを作る運用
```

`/blog` がない入力は記事化しません。

記事化しない入力には、必要に応じて次を使います。

```text
/private  記事化しない private な相談
/memo     自分用メモ。記事化しない
```

## プライバシー・安全性ルール

`draft: false` で記事を作成するため、記事本文には次を含めません。

- 会社内部情報
- 内部URL
- 非公開リポジトリ名
- 顧客名
- 個人名
- Slackや社内チャットの発言
- 認証情報、APIキー、トークン
- AWSアカウントIDなどの内部識別子
- 年収、住宅ローン、家族、健康、住所などの私的情報
- セキュリティレビューの具体的内容
- 未公開プロダクト仕様

社内固有の文脈が必要な場合は、公開可能な一般論に置き換えます。

```text
NG: 特定企業の予約管理でFeature Flagを導入する場合
OK: ある予約管理システムでFeature Flagを導入する場合
```

## このPC上での作業手順

### 1. 作業ブランチを作成する

```sh
git switch main
git pull
git switch -c add-note-YYYY-MM-DD-topic
```

既存の未コミット変更がある場合は、作業前に内容を確認します。自分が作成していない変更は勝手に戻しません。

### 2. Codexに記事作成を依頼する

依頼文は次の形式にします。

```text
/blog
以下の内容を元に、ブログ記事を作成してください。

テーマ:
...

要件:
- Article/note/YYYY/MM/ 配下にHTMLを作成
- HTMLメタデータコメントはastro-blogの同期形式に合わせる
- draft: false にする
- HTML/CSSの図解を含める
- 会社名、個人名、内部URL、顧客情報、認証情報、家庭・健康・金融情報は含めない
- 必要に応じて一般化する
- PR本文にプライバシーチェックリストを含める
```

### 3. 記事HTMLを追加する

例:

```text
Article/note/2026/06/2026-06-14-codex-blog-workflow.html
```

記事本文は日本語で、実務に使える内容を簡潔にまとめます。

注意点:

- 推測は推測として書く
- 事実確認が必要な内容は公開情報を確認する
- 公開情報の引用や参照が必要な場合は、一次情報を優先する
- 個別企業や個人に紐づく内容は一般化する
- HTML/CSSによる図解を最低1つ含める
- `https://lyricrime.com/` に寄せて、濃色背景・明色文字・明色ボーダー・余白重視・装飾控えめのデザインにする

### 4. ローカル確認を行う

最低限、追加ファイルと差分を確認します。

```sh
git status --short
git diff -- Article/note/
```

チェック観点:

- `draft: false` になっている
- `slug` とファイル名が対応している
- `createDate` と `updateDate` が正しい
- 禁止情報が含まれていない
- 画像リンクが壊れていない
- HTMLとして構造が壊れていない
- 図解がモバイル幅でも破綻しない

### 5. commitしてpushする

```sh
git add Article/note/YYYY/MM/YYYY-MM-DD-slug.html
git commit -m "Add note: article title"
git push
```

画像を追加した場合は、画像ファイルもcommitに含めます。

```sh
git add Article/note/YYYY/MM/YYYY-MM-DD-slug.html images/YYYY/MM/
git commit -m "Add note: article title"
git push
```

現在のブランチにupstreamがない場合は、初回だけ次のようにpushします。

```sh
git push -u origin add-note-YYYY-MM-DD-topic
```

### 6. blog_articleにPRを作成する

```sh
gh pr create
```

PR本文には次を含めます。

```md
## Summary

- Article/note/YYYY/MM/YYYY-MM-DD-slug.html を追加
- Codex生成記事として、公開前提の `draft: false` で作成

## Generated article path

- Article/note/YYYY/MM/YYYY-MM-DD-slug.html

## Privacy checklist

- [ ] Input started with `/blog`
- [ ] `draft: false`
- [ ] No company-internal information
- [ ] No internal URLs
- [ ] No private repository names
- [ ] No customer names
- [ ] No personal names from conversations
- [ ] No credentials, tokens, or API keys
- [ ] No account IDs or internal identifiers
- [ ] No salary, loan, family, health, address, or other private life details
- [ ] No security review findings
- [ ] No unreleased product details
- [ ] Content is generalized enough for public publication

## Verification

- [ ] Checked HTML diff locally
- [ ] Checked HTML structure
- [ ] Checked metadata comment
- [ ] Checked responsive diagram layout
- [ ] Checked privacy rules
```

## GitHub Actionsの同期方針

`blog_article` 側では、`main` にmergeされた記事変更をトリガーにします。

現在の `.github/workflows/post.yml` は、`Article/**` のpushを検知して `taiseimiyaji/astro-blog` へ `repository_dispatch` を送る構成です。

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'Article/**'
```

今後の運用では、少なくとも次のパスを同期対象にします。

```text
Article/**
images/**
```

`astro-blog` 側では、`repository_dispatch` を受け取って同期用ブランチを作成し、記事を `src/content/blog/Article/` へコピーしてPRを作成します。

```text
blog_article main
  -> repository_dispatch
  -> astro-blog workflow
  -> copy Article/note/** to src/content/blog/Article/note/**
  -> copy images/** if needed
  -> npm install / npm run build
  -> create sync PR
```

直接pushではなく、`astro-blog` 側にも同期PRを作成する方針にします。これにより、Astro側のビルド結果を確認してから公開できます。

## 必要なGitHub設定

`blog_article` 側に、`astro-blog` へ `repository_dispatch` を送れるトークンを設定します。

```text
Repository secrets:
  COPY_ACCESS_TOKEN
```

必要な権限の目安:

- `taiseimiyaji/astro-blog` への `repository_dispatch` 実行
- `astro-blog` 側でPRを作成する場合は、workflow側で利用するトークンにcontents/pull-requests権限

`astro-blog` 側には、`repository_dispatch` を受けるworkflowを用意します。

## 公開判定

この運用では、`draft: false` の記事をPRで追加します。

そのため、判定ポイントは次です。

```text
blog_article PR merge = 記事内容の承認
astro-blog PR merge = ビルド確認後の公開反映
```

レビューで不安がある場合は、PRをmergeせずに修正します。公開を止めたい場合は、`draft: true` に変えるのではなく、PRをmergeしないことを基本にします。

## 運用チェックリスト

記事作成時:

- [ ] 入力が `/blog` で始まっている
- [ ] `Article/note/YYYY/MM/` に追加している
- [ ] `draft: false` を設定している
- [ ] HTMLメタデータコメントが `astro-blog` の同期形式に合っている
- [ ] HTML/CSSの図解を含めている
- [ ] 禁止情報が含まれていない
- [ ] 社内固有情報を一般化している
- [ ] PR本文にプライバシーチェックリストを含めている

blog_article merge前:

- [ ] 差分をレビューした
- [ ] 公開してよい内容になっている
- [ ] ファイル名、slug、日付に問題がない

astro-blog merge前:

- [ ] 記事が `src/content/blog/Article/` 配下へ同期されている
- [ ] 画像が必要な場所へ同期されている
- [ ] Astro buildが成功している
- [ ] 表示確認に問題がない
