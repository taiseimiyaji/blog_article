---
title: ユースケースで選ぶSPAの認証——Laravel 12の Session / Sanctum とOIDC
tags: [laravel, auth]
createDate: 2025-08-11
updateDate: 2025-08-11
slug: laravel-auth
---

## まとめ

* **同一ドメインSPA × ブラウザだけ** → **Laravel Session（stateful）単独でも実装可能**。ただし**公式はSPAにはSanctum推奨**。
* **`routes/api.php` を保ったままSPAを stateful にしたい** or **同一トップレベルドメイン内の別サブドメインSPA** → **Sanctum（SPAモード）**。
* **別ドメインのSPA（例：`spa.example.com` と `api.other.com`）** → **Sanctum（SPA）不可**。

  * セッションで運用したいなら**BFF**で**同一サイト**に集約
  * もしくは**stateless**（**Sanctum Token** / **OAuth/OIDC**）へ
* **モバイル / 外部クライアント** → **stateless** が必要。**Sanctum（Token）** か **OAuth/OIDC**。
* **外部OIDCを使う** → **BFF（Code+PKCE→サーバ交換→アプリのセッション）**が基本。Sanctumは**必須ではない**。

## はじめに

今回はLaravelでSPAのWebアプリケーションを作成する際に利用できる認証機能を、ユースケース別の選び方という観点で整理します。Laravelには複数の手段があり、設計上の注意点も多いため混乱しがちです。そこで、要件から逆引きできる判断軸を提示します。

## なぜここで迷うのか

* Sanctumは**2つの機能**（APIトークン発行とSPA認証）を提供。**SPA認証はトークンを使わずセッション**を使う。
* **SPA認証は同一トップレベルドメイン前提**（サブドメインは可）。
* `auth:sanctum` は**自社SPAのCookieセッション**と**サードパーティのAPIトークン**を両方受けられる設計。
* Laravelは**XSRF-TOKEN Cookie**を発行し、**同一オリジン**ではAxios等が**X-XSRF-TOKEN**を自動付与。
* Laravelの**ブラウザ向け既定はセッションガード**。一方で**SPAやハイブリッドにはSanctum推奨**。

---

## 用語の確認

### stateful（セッション型）

* **概要**
  ブラウザに**Cookie（セッションID）**、サーバに**セッション状態**を持たせる認証。Laravelでは **`session` ガード**で実現。
* **どう成り立つか**
  ログイン成功 → **セッションに保存** → 以後はCookieの**セッションID**でユーザー復元。CSRFは**XSRF-TOKEN Cookie → X-XSRF-TOKEN**で防御（SPAではAxios等の自動付与を活用）。
* **利点**
  即時失効や権限変更の**即反映**が容易。長寿命トークンを置かないため**XSS耐性**が比較的高い。
* **注意点**
  **スケール時は共有セッションストア**（Redis等）。Cookie属性（SameSite/Secure/Domain）や**CORS**設計が必要。

### stateless（トークン型）

* **位置づけ**
  サーバ側にブラウザ専用状態を持たず、毎リクエストで**Bearerトークン**を検証。
* **どう成り立つか**
  クライアントが `Authorization: Bearer <token>` を毎回送信。サーバは署名/JWKs/DB照合で**都度認証**。
* **利点**
  **多クライアント**（モバイル/外部）向き。**マルチリージョン/エッジ**に載せやすい。
* **注意点**
  **失効・ローテーション・保管**の運用コストが高め。長寿命トークンの**漏えい対策**が要。

---

## Sanctum認証

### Sanctum（SPAモード）

* **概要**
  **自社SPAからのAPI**を**stateful（Cookie＋セッション）**として扱う。`auth:sanctum` はまず**認証Cookie**を確認し、無ければ**AuthorizationヘッダのAPIトークン**を確認。
* **実現方法（概略）**

  1. `config/sanctum.php` で **`stateful` ドメイン**を設定
  2. `bootstrap/app.php` で **`statefulApi()`** を有効化
  3. 初回に **`/sanctum/csrf-cookie`** でXSRF初期化
  4. 以降は**セッションCookie＋CSRF**で認証（Axiosは `withCredentials` / `withXSRFToken` を推奨）
* **要件・注意**

  * **同一トップレベルドメイン必須**。**サブドメインは可**だが、**異なるレジストラブルドメインは不可**。
  * リクエストには **`Accept: application/json`** と **`Origin` または `Referer`** を付与。
  * クロス**サブドメイン**でCookieを共有する場合は `SESSION_DOMAIN` やCookie属性・CORS設定を適切に。

### Sanctum（Tokenモード）

* **概要**
  GitHubのPATのような**APIトークン**を発行し、**Bearer**でAPI認証する**軽量トークン方式**（非JWTでも可）。
* **実現方法（概略）**

  1. ユーザーに `HasApiTokens` を付与
  2. `createToken()` で発行（**DBにはSHA-256でハッシュ保存**／**平文は発行時のみ表示**）
  3. ルートを `auth:sanctum` で保護
* **使いどころ**
  **モバイル/外部クライアント**や社内APIの**シンプルなトークン配布**に適合。**自社SPAの認証には使わず**、**SPAはSanctumのSPA機能**を使う。

---

## 分岐フロー

```
Q1: 自社ブラウザSPAだけ？
 ├─ Yes → Q2
 └─ No（外部/モバイルあり） → stateless要件 → Sanctum Token or OAuth/OIDC

Q2: 同一トップレベルドメイン内で出せる？
 ├─ Yes（同一サイト or サブドメイン）→ Laravel Session（最小） or Sanctum（SPA）※api.php維持なら推奨
 └─ No（別レジストラブルドメイン）→ Sanctum（SPA）不可
        → BFFで同一サイト化（セッション運用） or stateless（Sanctum Token / OAuth/OIDC）

補足: OIDC採用？ → BFFなら最終的にアプリのセッションで運用（Sanctum必須ではない）
```

## 迷いがちなポイント

* **「JS（SPA）ならSanctum必須？」** → **必須ではない**。**同一ドメインならセッションのみでも可能**。ただし**公式はSPAにはSanctum推奨**。
* **「自社SPA向けAPI」とは？」** → **自分たちが配信するブラウザSPAからの呼び出し**（第三者やモバイルは含まない）。
* **CSRFはどう効く？** → Laravelが**XSRF-TOKEN Cookie**を付与。**同一オリジン**ではAxios等が**X-XSRF-TOKEN**を自動付与。**クロスサブドメイン**では `withCredentials` 等の設定＋Sanctum側のstateful設定が必要。
* **即時失効を重視するなら？** → **stateful**が楽。`auth:sanctum` 運用でもCookie側で即時無効化しやすい。

## まとめ

* **誰が（自社/外部/モバイル）** × **どこから（同一サイト/サブドメイン/別ドメイン）** × \*\*運用要件（即時失効/拡張性）\*\*で選ぶ。
* 迷ったら：

  * **同一サイトSPA** → まずは**セッション**、`api`維持や拡張性重視なら**Sanctum（SPA）**。
  * **別レジストラブルドメイン** → **BFFで同一サイト化** or **stateless**（Sanctum Token / OAuth/OIDC）。
  * **多クライアント** → **stateless**前提（Sanctum Token / OAuth/OIDC）。

[1]: https://laravel.com/docs/12.x/sanctum "Laravel Sanctum - Laravel 12.x - The PHP Framework For Web Artisans"
[2]: https://laravel.com/docs/12.x/csrf "CSRF Protection - Laravel 12.x"
[3]: https://laravel.com/docs/12.x/authentication "Authentication - Laravel 12.x"
[4]: https://laravel.com/docs/12.x/session "HTTP Session - Laravel 12.x"
[5]: https://jetstream.laravel.com/features/api.html "API - Jetstream"
[6]: https://laravel.com/docs/12.x/passport "Laravel Passport - Laravel 12.x"