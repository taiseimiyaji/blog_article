---
title: Renovate + GitLab CI を使ってパッケージの依存関係を自動更新する
tags: [Renovate, GitLab, CI]
createDate: 2023-07-28
updateDate: 2023-07-28
slug: renovate-bot-for-gitlab
---

## Renovateとは

依存関係の自動アップデートツールです。

[公式サイトはこちら](https://docs.renovatebot.com/)

有名なのはdependabotですが、dependabotの場合は手動でalertsの内容を確認し、ローカルで`npm audit fix`や`composer update`を実行します。  
その後、修正内容をコミットしてプッシュする必要があります。  

ああ、めんどくさい。プログラマの三大美徳のうち「怠惰」だけでなんとかお仕事をしている僕としては自動化したい。  
そんなときに副業でお世話になっている会社の方から「Renovateいいよ」なんて話を聞きました。  
早速導入にチャレンジしてみます。  

公式サイトより

>Why use Renovate?  
>Get pull requests to update your dependencies and lock files  
>Reduce noise by scheduling when Renovate creates PRs  
>Renovate finds relevant package files automatically, including in monorepos  
>You can customize the bot's behavior with configuration files  
>Share your configuration with ESLint-like config presets  
>Get replacement PRs to migrate from a deprecated dependency to the community suggested replacement (npm packages only)  
>Open source  

DeepL翻訳

```
Renovateを使う理由
依存関係を更新し、ファイルをロックするためにプルリクエストを取得する
RenovateがPRを作成するタイミングをスケジューリングすることでノイズを減らす
Renovateはmonoreposを含め、関連するパッケージファイルを自動的に見つけます。
設定ファイルでボットの動作をカスタマイズできる
ESLintのような設定プリセットで設定を共有できます
非推奨の依存関係からコミュニティが提案する代替に移行するための代替PRを取得します(npmパッケージのみ)
オープンソース
```

サポートしているプラットフォームは以下の通りです。

>Supported Platforms  
>Renovate works on these platforms:  
>  
>GitHub (.com and Enterprise Server)  
>GitLab (.com and CE/EE)  
>Bitbucket Cloud  
>Bitbucket Server  
>Azure DevOps  
>AWS CodeCommit  
>Gitea and Forgejo  


## Renovateの導入

## GitHubの場合

GitHub上のOSSでもよく見かけるんですが、GitHubの場合はGitHub アプリとして実行できます。([参考](https://docs.renovatebot.com/modules/platform/github/))

## GitLabの場合

公式サイト: https://docs.renovatebot.com/modules/platform/gitlab/  
参考ブログ: https://panda-program.com/posts/renovate-gitlab  

### renovate.jsonの作成

細かい設定項目は公式ドキュメントを参照してください。  
https://docs.renovatebot.com/configuration-options/


私の場合はざっくり下記のような内容で作成しました。  
GitLab上で作成するMRに対してのラベルやReviewersを指定することもできます。  


```json
{
  "extends": [
    "config:base"            // 公式の推奨設定を継承して利用します
  ],
  "timezone": "Asia/Tokyo",
  "enabledManagers": [
    "npm",
    "composer"
  ],
  "lockFileMaintenance": {
    "enabled": true          // package-lock.jsonやcomposer-lock.jsonを更新するためのフラグです
  },
  "packageRules": [
    {
      "matchPackagePatterns": [
        "*"                  // 全パッケージを更新対象にします
      ]
    },
    {
      "matchDepTypes": [
        "dependencies"       // dependenciesのMRを作成します。
      ],
      "groupName": "dependencies"
    },
    {
      "matchDepTypes": [
        "devDependencies"    // devDependenciesのMRを作成します。dependenciesと分けて定義することで別々のMRが作成されるようになります
      ],
      "groupName": "devDependencies"
    }
  ],
  "ignoreDeps": [
    "@types/*",              // 更新対象外のパッケージを指定します
    "typescript",
    "php"
  ],
  "force": {
    "constraints": {
      "composer": "x.x.x",   // renovateの動作に使用するパッケージのバージョンを指定します
      "node": "x.x.x"
    }
  },
  "major": {
    "enabled": false         // メジャーバージョンの更新を禁止します
  },
  "baseBranches": [
    "master"                 // 更新対象のブランチを指定します
  ],
}
```

### Dockerを使用してRenovateを動かす

Renovateでは公式でDockerイメージを提供しています。  
今回の導入方法では、Dockerを使用してRenovateを動かします。  

https://hub.docker.com/r/renovate/renovate

設定作業を行う際には、ローカルで動作確認をしながら設定項目を追加していくのがいいと思います。

```shell
docker run --rm -e GITHUB_COM_TOKEN={{ GitHubのpublic repo権限のトークン。リリースノートの取得に使用 }} renovate/renovate:xx.xx.xx renovate --platform gitlab --token {{ GitLabのAPI_TOKENの値 }} --endpoint https://git.plugram.co.jp/api/v4 smaregi/dev-site
```

### GitLab CIで設定する

```
"Renovate Bot":
  stage: {{ 任意のステージ }}
  image:
    name: renovate/renovate:xx.xx.xx // 任意のバージョンを指定してください
    entrypoint: [""]
  script:
    - renovate --platform gitlab --token $API_TOKEN --endpoint $CI_SERVER_URL/api/v4 $CI_PROJECT_PATH
  only:
    - schedules  // 設定例ですが、定期実行の設定をしておくといいです
```

CIで記載する場合は各環境変数をリポジトリに設定しておく必要があります。  
筆者の環境だと`Setting`>`CI/CD`>`Variables`から設定できました。  

CI上で動作させる場合、環境変数GITHUB_TOKENはrenovateがデフォルトで参照するので`Settings`>`CI/CD`>`Variables`に設定できていれば大丈夫です。  
ローカルの場合は先程のコマンドの通り`-eオプション`で渡すようにしてください。


## ハマったポイント

### package-lock.jsonの更新差分が多すぎる

原因は使用しているImageが古く、プロダクトが使用しているNode.js、及びnpmのバージョンとrenovateが動作するNode.js、及びnpmのバージョンが異なっていたためでした。

https://docs.renovatebot.com/configuration-options/#constraints

lockFileVersionについては少し話がそれてしまうのでChatGPTに説明してもらいます。

```markdown
npmの`lockFileVersion`は`package-lock.json`ファイルに含まれ、npmが使用するロックファイルのバージョンを示します。このバージョン情報はnpmがパッケージの依存関係をどのように解決・記録するかを決定します。

`lockFileVersion`の違いは以下の通りです：

- `lockFileVersion: 1`: これはnpm v5から導入されました。`package-lock.json`には、プロジェクトでインストールされている各パッケージの具体的なバージョンとその依存関係が記録されています。

- `lockFileVersion: 2`: これはnpm v7で導入されました。これはv1と同様の情報を含んでいますが、加えて`optionalDependencies`（オプションの依存関係）を考慮に入れた形式になっています。このフォーマットは、従来のnpmと互換性があり、また、Yarnでも利用することが可能です。

- `lockFileVersion: 3`: これもnpm v7で導入されました。v2と同様の情報を含みつつ、`peerDependencies`（ピア依存関係）をより適切に表現できるように改良されています。ただし、このフォーマットはYarnと互換性がありません。

これらの`lockFileVersion`の目的は、開発者間でのパッケージのバージョン管理を容易にし、それぞれが同じ依存関係を持つことを保証することです。これにより、"開発環境で動作するが、プロダクション環境で動作しない"といった問題を避けることができます。

また、これらのバージョンは、特定のnpmのバージョンでデフォルトとして生成されますが、他のnpmのバージョンでも問題なく解釈・利用できます。たとえば、npm v7はデフォルトで`lockFileVersion: 2`または`3`のロックファイルを生成しますが、それでもnpm v5やv6はこれを正しく解釈できます（ただし、`lockFileVersion: 3`の一部情報は無視されます）。
```

とはいえ、互換性があろうがなかろうがrenovateが作成する`package-lock.json`のバージョンと手動で`npm install`を実行したときのバージョンが異なると、`package-lock.json`の更新差分が多くなるので避けたいと判断しました。

公式サイトを参考に、下記のような形で指定することで解決しました。

```json
{
  "force": {
    "constraints": {
      "node": "< 15.0.0"
    }
  }
}
```

>Constraints are also used to manually restrict which datasource versions are possible to upgrade to based on their language support. For now this datasource constraint feature only supports python, other compatibility restrictions will be added in the future.

と記載があり、`Pythonにしか対応していないのか?矛盾してるじゃないか` と思っていましたが、どうやら、言語自体のバージョンを固定する際に使用するようです。Python2系とPython3系を明示的に制約をかけたい場合は多いと思うのでそのための記述ですね。

筆者の環境では、この設定で無事npmのバージョンを意図したもので動作させることができましたが、CIに使用するImageサイズを下げたい等を実現する場合はDockerfileを自前で用意する必要があります。  
DockerHubやGitLab Container Registry等を使用してCIから利用できる形にしておくといいと思います。

## 導入してみた感想

導入してみるまでは単なる`npm install`とpushを自動化するツール、くらいの感覚でいましたが、想像以上にメリットが大きかったです。

具体的には、MRに以下のような内容を含めてくれます。

- アップデートするライブラリのバージョン間のリリースノートをまとめてくれる
  - 例えば、`1.0.0 => 1.2.0`の場合はその間に含まれるバージョンのリリースノートをまとめてくれます。
  - また、バージョン間のライブラリ自体のコード差分も生成してくれます。
- アップデートするライブラリのドキュメントへのリンク、およびGitHubへのリンクが含まれる
  - パッケージアップデートによる不具合発生時にissue探すときに楽になります
- アップデートするライブラリの単位を指定できる
  - dependenciesはMRをレビューを確認し、devDependenciesはauto mergeにする等、自由に設定できます。

また、言語自体のアップデートも行えるため、Node自体であったり、PHP自体のアップデートも行えます。

コンテナ化されたプロジェクトの場合はより恩恵を受けられると思います。

自動化自体も好きですが、自動化にとどまらず、リリースノートを都度確認する習慣ができ、依存パッケージへの意識を高めてくれるツールだと感じました。  
書籍「ソフトウェア設計のトレードオフと誤り」では、`あなたが使うライブラリはあなたのコードとなる`という章が存在するくらいなので、この意識は持ち続けたいですね。  