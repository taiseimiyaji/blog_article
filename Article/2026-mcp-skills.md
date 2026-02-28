---
title: MCPやAgent Skillsあれこれ
tags: [ai]
createDate: 2026-02-28
updateDate: 2026-02-28
slug: 2026-mcp-skills
---

## はじめに

生成AIを使った業務が当たり前になってしばらく経ちますが、たまにMCPやAgent Skillsを知らない人や活用できていない人を見かけるので、関連する公式ドキュメントをまとめつつ、簡単に解説した記事を書いておきます。  

仕組みとして理解していても、ローカルMCPとリモートMCPの違い、Agent Skillsとの違いや認証情報の扱い、サードパーティ製ツールの利用の際の注意など意外と気をつけないといけないことが多いのでその辺りにも触れます。

また、社内でこれらの仕組みを活用するにあたり、Anthropic発でコミュニティや複数ベンダーが扱えるMCPの考え方と、Claude Code 向けMarketplaceの活用についてまとめます。

## 1. MCPとは何か

MCP（Model Context Protocol）は、モデル（LLM）が外部のツールやデータソースを使うための“共通の接続方式”を提供する仕組みです。外部連携を「個別実装の寄せ集め」にせず、標準化された形で安全に扱えるようにすることが主目的です。

元々AIツールでは OpenAI のFunction Callingなどで外部呼び出しを行う方法は存在していました。MCPはそれを完全に置き換えるというより、外部接続の方式を共通化しやすくする「補完的な標準化アプローチ」と捉えるのが実務的です。

MCPはAnthropic発の取り組みですが、仕様自体はコミュニティや複数ベンダーが扱える形で公開されています。

### MCPの目的

* モデルが外部ツールを安全に使えるようにする
* ツール連携のI/Fを共通化し、クライアント（AI側）とサーバ（ツール側）の責務を分離する
* 接続先や認可の差し替えを前提にした拡張性を持たせる

### 何が嬉しいのか

* **ツール連携の標準化**：AIアプリが一度MCPに対応すれば、同じ作法でさまざまなツールに繋げられる
* **権限分離**：ツールごとに権限・監査・ネットワーク境界を切りやすい
* **個人権限**: PATを発行できるシステムでは、自分の権限で操作できる
  * 利点: 操作主体の監査や責任追跡がしやすい
  * 注意点: 持ち出し、権限過多、退職時の失効漏れリスクがある

## 2. Agent Skillsとは何か

Agent Skillsは、エージェントが作業を再現性高くこなすための「作業レシピ／手順の型」です。人が毎回プロンプトを手で組み立てなくても、一定の品質と手順でタスクを回せるようにするための仕組みとして捉えると分かりやすいです。

MCPは個別ツールへの接続方式、Skillsは再利用可能な手順定義、と言うと整理しやすいです。実行そのものはエージェント（モデル+ツール実行基盤）が担います。

### Skillsの位置づけ

* **作業レシピ**：やるべき順番、注意点、出力フォーマット、チェック観点などをまとめる
* **品質の標準化**：属人化しがちな“良いプロンプト”をテンプレ化して配布できる
* **運用のしやすさ**：手順がコードやドキュメントに固定されるのでレビューしやすい

### MCPとSkillsの関係（接続と手順の分離）

* MCPは「どこと繋ぐか」「何ができるか（接続I/F）」
* Skillsは「繋いだ先をどう使って目的を達成するか（ワークフロー）」

この分離により、接続先（ツールやデータ）の置き換えが起きても、手順（Skills）を大きく変えずに済むことが多いです。

## 3. ローカルMCPとリモートMCPの違い

同じMCPでも「どこで動かすか」によって、責任範囲・セキュリティ境界・運用コストが大きく変わります。ここを曖昧にすると、認証情報や監査の設計で詰まりやすいです。

### 実行場所・接続方式・認証・ネットワーク境界

設計時は、次の4軸で並べると判断しやすくなります。

* 接続方式: ローカル標準入出力（stdio）中心か、HTTPなどのネットワーク経由か
* 実行主体: 利用者端末で起動するか、社内/クラウドの共通基盤で起動するか
* 認証: OAuth / PAT / サービスアカウントのどれを使うか
* ネットワーク境界: 社内NW限定か、ゼロトラスト/VPN経由か、公開境界を跨ぐか

* **ローカルMCP**：開発者端末や利用者端末でMCPサーバを動かす（またはローカル資産にアクセスする）

  * 端末内のファイルやローカルDB、ローカルネットワーク資産へアクセスしやすい
  * ただし認証情報が端末に散らばりやすく、統制が難しくなりがち

私が作成したMCPサーバーとしては、togglのMCPサーバーがあります。

https://github.com/taiseimiyaji/toggl-mcp-server

配布方法として、Docker Imageを配布してローカルで動かしてもらうか、npmパッケージとして公開してインストールしてもらいローカルで動かしてもらう方法の二つがあります。

作成自体はかなり簡単で、APIがあるものならほとんど流用できるものが多いです。Dockerで配布する場合はGoやRustなど実行ファイルが軽い状態で配布できる言語を選択するのもありだと思います。

* **リモートMCP**：社内サーバやクラウド上でMCPサーバを運用する

  * 認証情報を集中管理しやすい（Secrets管理や監査ログの集約など）
  * ネットワーク境界を明確にできる一方、運用・申請・セキュリティ対応のコストがある

リモートMCPはローカルPCではなく、クラウド上などで動作するMCPサーバーになります。参考例として、GitHub が提供しているものを紹介します。

https://github.com/github/github-mcp-server

ローカルとは違い、PC上で動かす必要はなく、MCP設定ファイルにOAuthやPATなどの認証設定を書いて利用します。

OAuthを使うと資格情報の配布/管理負荷は下がりやすいですが、トークン保管先、リフレッシュ、スコープ、同意画面、失効設計は引き続き必要です。

また、GitHub操作のようにCLIが十分強力なケースでは、MCP経由よりCLI直利用の方が文脈転送量と失敗要因が少ない場合があります。例えば「単一リポジトリでIssueを1件作る」程度ならCLIが速く、「複数ツールを横断した要約・判断」が必要なときはMCPの価値が出やすい、という切り分けです。

### “ローカルで動かすべきもの / リモートに寄せるべきもの”の判断軸

* **ローカル寄りが向くケース**

  * 個人の開発効率が主目的で、機密データに触れない
  * PoCで早く検証したい
  * 利用者ごとに環境差があっても許容できる

* **リモート寄りが向くケース**

  * 機密データ・個人情報・顧客データに触れる
  * 組織として配布・統制したい
  * 監査・DLP・許可リスト・レート制限などを入れたい

## 4. MCPとAgent Skillsの違いを図で整理

MCPとSkillsを混同しないために、「コネクタ」「ワークフロー」「ポリシー」の3点セットで整理するのがおすすめです。

* **コネクタ（MCP）**：外部ツールやデータへの接続口
* **ワークフロー（Skills）**：作業手順・テンプレ・チェック観点
* **ポリシー（ガードレール）**：認可、監査、DLP、利用制限、禁止事項

### 図（概念）

![MCPとSkillsの関係図（MCP=接続、Skills=手順、Policy=制御）](/images/image-1.png)

画像が表示されない場合は、次の簡易図で読み替えてください。

```text
[Agent]
  |- MCP (Connector) -> [Tool/Data]
  |- Skills (Workflow) -> [手順定義]
  '- Policy (Guardrail) -> [認可/監査/DLP]
```

### よくある誤解

* **Skillsだけで連携できる？**
  Skillsは手順であり、実際の接続や操作I/FはMCPやAPIクライアント等が別途必要です。

* **MCPを入れれば安全？**
  MCPは標準化の枠組みであり、安全性は認可・監査・ネットワーク境界・Secrets運用などの設計に依存します。

## 5. サードパーティ製ツールをつなぐときの注意点

サードパーティ連携は便利ですが、社内の業務データと混ざった瞬間にリスクが跳ね上がります。最低限「繋ぐ前に見るチェックリスト」を用意しておくと、導入が速くなり、事故が減ります。

### データ持ち出し・学習利用・ログ保存

* 送信データが学習に使われるか（オプトアウト可否含む）
* ログ保存期間、再利用、第三者提供の有無
* 機密データ・個人情報の扱い

これらのMCPサーバー自体の実装の安全性を自分で確認できない場合は利用を避けましょう。会社で許可されているMCPサーバーのみを利用するなど、判断できる人の許可があるとより良いと思います。

### 最低限のチェックリスト

* データの保管場所と保存期間は明確か
* 学習利用の有無とオプトアウト可否は明示されているか
* 送信対象の制御（DLP/マスキング）を設定できるか
* 権限スコープと監査ログの取得範囲は十分か
* 供給元の信頼性（署名、SBOM、脆弱性対応方針）を確認できるか

## 6. Claude Code Marketplaceを社内でどう活用するか

私は立場上生成AIに関するツールを整備したり、提供したりする仕組みを考えることが多いですが、Anthropicがすでに公式で提供してくれています。

Claude CodeのMarketplaceは、プラグイン（拡張）を配布・導入・更新するための仕組みです。

これを活用すると社内展開で重要な「便利な拡張を増やす」ことと「審査と更新の導線を作る」ことを同時に達成できます。

### Marketplaceリポジトリの社内公開

Marketplaceは、ドキュメント通りGitリポジトリとして公開すると、cloneする権限のある人全員が簡単に利用できるようになります。

中身はGitリポジトリなので、社内展開しておくと、各々好きなskillsなどのプラグインを作成し、簡単にPRを作成することができます。

リポジトリ管理者として、生成AIに詳しい人やシニアエンジニアなどを置いておくと、適切にレビュープロセスを構築することができ、簡単に運用できるようになると思います。

社内全体でなく、チームごとに作成してもいいかもしれません。

## 7. まとめ

* MCPは「外部ツール連携を標準化するための接続口」で、Function Callingと役割が重なる部分を補完する
* Skillsは「作業手順を標準化するためのレシピ」で、必要な道具を使う手順を共通化、再現しやすくする
* ローカル/リモートの選択は、セキュリティ境界と運用コストの選択
* 社内展開で重要なのは「接続・手順・ポリシー」をセットで運用すること
* Marketplaceは、社内で拡張を広げるための“審査と更新の仕組み”として活用しやすい

### 参考文献

* [https://www.anthropic.com/news/model-context-protocol](https://www.anthropic.com/news/model-context-protocol)
* [https://modelcontextprotocol.io/](https://modelcontextprotocol.io/)
* [https://modelcontextprotocol.io/specification/](https://modelcontextprotocol.io/specification/)
* [https://github.com/modelcontextprotocol/modelcontextprotocol](https://github.com/modelcontextprotocol/modelcontextprotocol)
* [https://platform.claude.com/docs/ja/agents-and-tools/agent-skills/overview](https://platform.claude.com/docs/ja/agents-and-tools/agent-skills/overview)
* [https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
* [https://www.anthropic.com/engineering/code-execution-with-mcp](https://www.anthropic.com/engineering/code-execution-with-mcp)
* [https://code.claude.com/docs/ja/plugin-marketplaces](https://code.claude.com/docs/ja/plugin-marketplaces)
* [https://code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview)
* [https://github.com/anthropics/claude-code/blob/main/plugins/README.md](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
