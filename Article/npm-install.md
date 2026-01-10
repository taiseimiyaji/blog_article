---
title: npm install は必ずしも package-lock.json を書き換えない
tags: [npm]
createDate: 2026-01-09
updateDate: 2026-01-09
slug: npm-install
---

## はじめに

`npm install` と `npm ci` の違いを「package-lock.json を書き換えるかどうか」として認識していました。

誤解の内容: 

- `npm install`はpackage-lock.jsonを書き換えて最新の依存関係でインストールする
- `npm ci`はpackage-lock.jsonを参照して依存関係をインストールするのでpackage-lock.jsonを書き換えない

結論から言うと、`npm ci` は **「絶対に lockfile を書き換えない」** のに対して、`npm install` は **「条件次第で書き換える（が、常に書き換えるわけではない）」** です。

もっというと、package-lock.jsonがある場合は先に確認し、package.jsonとの整合性を確認します。整合性が取れていない場合にpackage-lock.jsonを書き換えるか、エラー終了するかがコマンドによって異なります。

[npm-ci | npm Docs](https://docs.npmjs.com/cli/v11/commands/npm-ci)

## npm ci の定義（公式）

公式ドキュメントでは `npm ci` は「CI/自動環境向けのクリーンインストール」、「依存関係のクリーンインストール」という位置づけです。

* 既存の `package-lock.json`（または `npm-shrinkwrap.json`）が必須
* `package.json` と lockfile が一致していない場合、lockfile を更新せずエラー終了
* 既存の `node_modules` があれば 削除してから インストール
* `package.json` も “あらゆる package-lock.json” にも一切書き込まない

### npm-shrinkwrap.json とは

npm-shrinkwrap.jsonはざっくりいうとパッケージの公開時にロックファイルとして使用されるファイルです。

このファイルが存在している場合、package-lock.jsonが無視されて、npm-shrinkwrap.jsonが優先されます。

パッケージを公開するときにロックファイルが含められるので、ユーザー側での依存関係の変更ができないようになります。

特殊なユースケースでの利用が考えられますが、基本的にpackage-lock.jsonを利用するべきです。

## npm install の定義（公式）

一方 `npm install` は、依存関係のインストールだけでなく **lockfile の生成/更新にも関わる** コマンドです。
公式の `package-lock.json` の説明にはこうあります。

* `package-lock.json` は、npm が `node_modules` ツリー か `package.json` を変更する操作で自動生成される
* 古い lockfile（npm v6 以前など）を検出すると、インストール過程で 不足情報を埋めるために自動更新される

ここが重要で、**`npm install` は “必要があれば” lockfile を更新しますが、必要がなければ更新しないこともあり得る**、という理解になります。

## npm install でも書き換えない場合

### 1) 依存ツリーが変わらない（＝lockfile更新が不要）

`package-lock.json` は「npm が `node_modules` ツリーや `package.json` を変更したときに生成される」という定義です。
逆に言えば、**結果としてツリーやメタ情報の差分が発生しないケースでは、lockfile が更新されない**ことがあります。

実務的には、例えば

* すでに `node_modules` が期待通りで、追加/更新が起きない
* lockfile が現行フォーマットで、補完が不要

のようなときに「`npm install` したのに lockfile が変わらない」可能性があります。

### 2) `--no-save` / `save=false` にしている

そもそも意図的に設定でコントロールすることができます。

公式ドキュメント では `save=false`（≒ `--no-save`）の場合、“package-lock.json を書かない” と書かれています。

### 3) `package-lock=false`（または `--no-package-lock`）にしている

同様に変更しない設定が他にもあります。

公式ドキュメント では `package-lock` 設定が false の場合、lockfile は無視され、書き込まれないという記述もあります。

## CI ではどっちを使うべきか

npm の公式説明の通り、**CI/自動環境は基本 `npm ci` が想定**です。

ややこしいのはciというのはエイリアスからもわかるようにclean installの意味合いだと思われるんですが、継続的インテグレーションの意味のCIとかぶってしまっているのでややこしいですね。

まあ勘違いしたとしても正しい使い方がされるので困ることはなさそうですが。

* CIで「同じ依存を確実に入れたい」→ `npm ci`
* ローカルで「依存を追加/更新し、lockfile を更新したい」→ `npm install`（必要なら `npm install <pkg>`）

という使い分けになります。

## 直近の変更・注意点（npm v11.6.2 以降の “in sync” 問題）

「直近で変わった/荒れている挙動」として、**npm v11.6.2 前後で `npm ci` が “package.json と lockfile が in sync ではない” として失敗する** という報告が複数あります。 

[[BC BREAK] version 11.6.2 breaks CI · Issue #8669 · npm/cli](https://github.com/npm/cli/issues/8669)

この手の問題は、`npm ci` の定義そのものが変わったというより、**lockfile 生成側（install/update）との整合が崩れて “ci が厳密に見て落とす”** 挙動になるようです。

[[BUG] `npm ci` fails because `npm install` produces an out-of-sync lockfile (regression since v7.0.9) · Issue #8726 · npm/cli](https://github.com/npm/cli/issues/8726)

**現実的な対策**：

* CI で使う npm を **明示的に固定**する（Node 同梱 npm をそのまま使わず、toolchain 管理する）
* lockfile を生成したときのフラグ（例：`--legacy-peer-deps` 等）を CI 側でも揃える
* `npm ci` が落ちたら、まずはローカルで `npm install` → 差分が出るか確認し、意図せぬ設定（`save=false`, `package-lock=false`）が混ざっていないか確認

## npm豆知識

公式ドキュメントを眺めていて、なんとなくの挙動は予想していたものの、ドキュメントでしっかり確認していなかった部分をまとめます。

### Git上のリポジトリのインストールにおけるGit Clone

通常のレジストリのパッケージの場合は範囲指定しますが、これらは設定されたレジストリから解決されます。

実態としては「gzipped tarball(.tgz)」をダウンロードして展開する形式です。

tarball URLを直接指定することもできます。

Git URLの場合は少し特殊で、`git clone`を行って取得する形式です。

末尾に`#<commit-ish>`を指定することで、特定のコミットを指定することもできます。

`#semver:<range>`という形式だとリモートのtagを探索することもできます。

また、サブモジュールがある場合はそちらも一緒にcloneされます。

### devDependencies の扱いについて

NODE_ENVがproductionの場合にインストールされない認識だったdevDependenciesですが、実際には設定のomitの値がNODE_ENV依存になっているようです。

omit=devの場合、devDependenciesを無視するようになっており、このデフォルト値がNODE_ENVの値によって変わります。

環境変数によらずインストールを抑制したい場合には `npm ci --omit=dev` を指定することで抑制できます。

### 依存関係の調査に `npm explain` が便利

`npm explain <package>` を実行することで、依存関係の詳細を確認できます。

どの経路で依存関係に含まれたかを説明してくれるため、トラブルシュートに使えそうです。

## まとめ

今回はnpm install周りの挙動について調査しました。

案外公式ドキュメントを読まないままタスクランナーなどを利用するだけになっている現場も多いと思うので、一度公式ドキュメントを読んでみると意外な発見があるかもしれません。
