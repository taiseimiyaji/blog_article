---
title: LaravelのセッションとReact Queryで踏みがちなレースコンディションの話
tags: [Laravel, React Query]
createDate: 2025-11-30
updateDate: 2025-11-30
slug: laravel-session-race-condition
---

## はじめに

「POSTでセッションに値を保存したはずなのに、すぐ後のページで `null` になっている」

セッションを使ってページをまたぐデータの受け渡しなどを実装していると、たまにこんな **「保存したはずの値が消える」現象** に出会います。

しかもログを見ると：

  * 保存APIの中ではちゃんと値が入っている
  * 直後にセッションから再取得してもちゃんと入っている
  * なのに、次のページのGETでは `null` になっている

という、かなり不可解な状態になります。

この記事では、実際に遭遇したケースを元に、

  * Laravelのセッションミドルウェアがどう動いているか
  * **GETエンドポイントでもセッションが保存される** という落とし穴
  * React Queryの楽観的更新（`onMutate`）と依存クエリの組み合わせでどうレースが発生するか
  * どう防ぐ・緩和するか

についてまとめます。

-----

## 1\. 何が起きていたか

現象を抽象化すると、以下のようなフローで発生していました。

1.  ページAで、セッションに「状態オブジェクト」を保存するAPI（`PUT /state`）を叩く
2.  保存が成功したので、ページBに遷移する
3.  ページBは初期表示で `GET /state` を叩き、セッションから状態を読む
4.  **このとき、保存したオブジェクトの一部（例：`updated_value`）が `null` になっている**

API側のログを見ると：

  * `PUT /state` の中では受け取った値は正しい
  * セッションに `put()` → 直後に `find()` しても値は入っている
  * しかし、その後の `GET /state` では同じキーの値が `null` になっている（変更前に戻っている）

つまり、

> 「保存 → 再取得」まではOKなのに、**その後どこかでセッションの値が巻き戻されている**

ように見えます。

-----

## 2\. Laravelのセッションミドルウェアの動き

まず、Laravelの `StartSession` ミドルウェアがどう動いているかを整理します。

ざっくり言うと、各リクエストで以下の処理が行われます。

1.  **リクエスト開始時**
      * セッションハンドラからデータを読み込み、メモリ上（`$request->session()`）に展開する
        （ファイルドライバなら `read(session_id)` → `unserialize` した配列）
2.  **アプリケーション処理**
      * コントローラやミドルウェアが `session()->get()` / `session()->put()` する
3.  **リクエスト終了時**
      * メモリ上のセッション状態をまとめて `write(session_id, serialized_value)` する

重要なのは、

> **「最後に終わったリクエストのセッション状態が“正”としてストレージに残る」**

という性質です。

### 2.1 GETでもセッションは保存される

ここが今回のキモですが、

  * コントローラが何も `session()->put()` していない **ただのGETリクエストであっても**
  * リクエスト開始時点のセッションを読み込み、
  * リクエスト終了時に、**そのまま**ハンドラに `write()` します

つまり、

> 「読むだけのつもりのGETリクエスト」が、結果的に **古いセッション状態をストレージに書き戻す（上書きする）** ことがあります。

-----

## 3\. レースコンディションのタイムライン

次に、実際に起きていた競合状態（レースコンディション）をタイムラインで追ってみます。

### 3.1 2本のリクエストが並行するとどうなるか

例として、以下の2つのリクエストがほぼ同時に走るとします。

  * **Request A**: `PUT /state`（状態を更新するAPI）
  * **Request B**: `GET /something`（別リソースの一覧取得APIなど）
      * コントローラ内ではセッションを読むだけだが、`StartSession`は有効

これらは同じ `session_id` を持っています。

#### タイミング例

1.  **Request B (GET) 開始**
      * 開始時点のセッション：`state.value = null`
      * ミドルウェアがこの状態をメモリに読み込む
2.  **Request A (PUT) 開始**
      * 開始時点ではまだ `state.value = null`
      * アプリケーション内で `state.value = "new_value"` に更新し、処理を終える
3.  **Request A 終了**
      * `StartSession` がメモリ上のセッション（`value = "new_value"`）を書き戻す
      * この時点ではストレージ上も `"new_value"` に更新されている
4.  **少し遅れて Request B 終了**
      * しかし、Request B が持っているメモリ上のセッションは **開始時点のまま（`value = null`）**
      * `StartSession` は「このリクエスト（B）のメモリ上のセッション」をストレージに書き戻す
      * 結果、ストレージ上の値が **`value = null` に巻き戻る**

このあと `GET /state` を叩くと、
「PUTで更新したはずの `value` がnull」になって返ってくる、というわけです。

-----

## 4\. React Queryとの組み合わせでなぜ頻発したか

ここまでの話だと、
「いやでもそんな都合よく並列リクエストなんてそうそう起きないのでは？」
と思うかもしれません。

今回のポイントは **React Queryの楽観的更新（`onMutate`）と依存クエリ** の組み合わせでした。

### 4.1 状態保存mutationと楽観的更新

状態オブジェクトを更新するmutationを、React Queryで次のように書いていたとします。

```ts
const useStateMutation = () => {
    const queryClient = useQueryClient();
    const key = ['state'];

    return useMutation({
        mutationKey: ['state-mutation'],
        mutationFn: (payload) => api.put('/state', payload),

        onMutate: async (next) => {
            // 1. 既存のクエリをキャンセル
            await queryClient.cancelQueries({ queryKey: key });

            // 2. 現在の値を退避
            const prev = queryClient.getQueryData<State | null>(key) ?? null;

            // 3. 🔴 サーバレスポンスを待たずにキャッシュを書き換える
            queryClient.setQueryData(key, next);

            return { prev };
        },
        // onError, onSuccess, onSettled ... など
    });
};
```

  * ユーザーが操作して `mutate` を呼ぶ
  * `onMutate` で **即座にキャッシュを書き換える** ため、
      * コンポーネントから見ると「保存前から state が新しい値に変わっている」ように見えます

### 4.2 依存クエリがSnapshotをキーにするパターン

別のコンポーネントが、`state` を元に何かの一覧を取得しているとします。

```tsx
const SomeComponent = ({ state }: { state: State | null }) => {
    // stateの中身が変わるとキーが変わる
    const snapshotKey = JSON.stringify(state?.condition ?? {});

    const { data } = useQuery({
        queryKey: ['items', snapshotKey],
        queryFn: () => api.get('/items', { params: { /* state に依存した条件 */ } }),
    });

    // ...
};
```

こういう構成だと、

  * `state` が変わる → `snapshotKey` が変わる
  * → React Query が `['items', snapshotKey]` のクエリを新規に発行
  * → **新しい条件で `/items` GET が飛ぶ**

ここで問題になるのは、

> `state` が変わるのが「サーバ保存後」ではなく、
> **「`onMutate`タイミング（サーバ保存前）」である**

という点です。

### 4.3 結果として発生していたこと

これらを組み合わせると、以下のような流れになります。

1.  画面で保存アクションを実行する
2.  mutation が発火し、`onMutate` でローカルの `state` キャッシュが即座に更新される
3.  それを参照している別コンポーネントの `snapshotKey` が変わる
4.  React Query が `/items` の GET を即座にフェッチしに行く
5.  **この `/items` GET と、保存処理の `/state` PUT が、サーバ側では同一セッションで並行する**
6.  先ほどのセッションのタイムライン通り、
      * PUTで更新したセッションが、
      * あとから終わったGETにより **古い状態で書き戻される**

これが「保存したはずなのに、次の画面でnull」が起きていた正体でした。

-----

## 5\. 対策案（フロントエンド）

### 5.1 楽観的更新の使いどころを絞る

少なくとも「セッションで保持する状態オブジェクト」を更新するmutationについては、

  * **`onMutate`で `setQueryData` しない**
  * サーバ保存完了後にだけキャッシュを更新する

という設計が安全です。

```ts
const useStateMutation = () => {
    const queryClient = useQueryClient();
    const key = ['state'];

    return useMutation({
        mutationKey: ['state-mutation'],
        mutationFn: (payload) => api.put('/state', payload),

        // onMutate は使わない or ローカルUIの表示制御だけにとどめる
        onSuccess: (data) => {
            // サーバが確定させたstateだけをキャッシュに反映
            queryClient.setQueryData(key, data.state);
        },
    });
};
```

**ポイント：**

> 「QueryKeyに影響する値」は、**サーバ確定後のものだけを使う**

ようにすると、「サーバ保存前に依存クエリのGETを飛ばしてしまう」パターンを避けやすくなります。

### 5.2 依存クエリの `enabled` をきちんと設計する

今回のように、

  * 裏側でコンポーネントはマウントされている
  * でも「今この瞬間、この一覧取得は不要」というケース

では、`useQuery` に `enabled` を付けるのが有効です。

```ts
const useItems = (params: Params, enabled: boolean) => {
    const snapshotKey = JSON.stringify(params);

    return useQuery({
        queryKey: ['items', snapshotKey],
        queryFn: () => api.get('/items', { params }),
        enabled, // false の間はfetchもrefetchもしない
    });
};
```

「いつフェッチするべきか？」「いつは絶対にフェッチしなくてよいか？」を意識して `enabled` を設計すると、**不要なGETリクエスト自体を抑制** でき、セッション巻き戻りのリスクを減らせます。

-----

## 6\. 対策案（バックエンド）

フロント側を直せば今回のような症状はかなり減りますが、理屈の上では「別タブ」や「複数リクエスト」が同時に走れば、同種の問題はやはり起こり得ます。

### 6.1 セッションから専用テーブルへの移行

根本的に安全側に寄せるなら、

  * 「進行中の状態」をセッションではなく **専用テーブル（DB）** で管理する

という選択肢が確実です。

  * テーブル例：`draft_states`（ユーザーID + 状態種別などをキーにする）
  * 楽観ロック：
      * `version` カラムを持たせ、`version` を条件に `UPDATE` する
  * 悲観ロック：
      * 重要な更新だけ `SELECT ... FOR UPDATE` してから更新

これなら、

> 「最後に終わったHTTPリクエストが正」という
> セッション特有の挙動から切り離せる

のが最大のメリットです。

### 6.2 Laravelの Session Blocking の活用（オプション）

Laravelには、「同一セッションのリクエストを直列化する」ための機能（Session Blocking）があり、対応ドライバ（RedisやMemcachedなど）を使っていれば利用できます。

これを使うと、同一セッションIDに対してリクエストが **同時に走らなくなる（ロック待ちになる）** ため、「古いGETがあとからセッションを書き戻す」という問題は物理的に起きにくくなります。

ただし、

  * 全ルートに適用するとパフォーマンスに影響が出る可能性がある
  * どのルートに適用するかの設計が必要

といったトレードオフはあります。

-----

## 7\. まとめ

今回の話を一文でまとめると、

> **「セッション + 並列リクエスト + React Queryの楽観更新」は、思った以上に簡単にレースコンディションを起こす**

ということです。

  * Laravelの `StartSession` は **GETでもセッションを書き戻す**
  * 同一セッションで複数リクエストが並列すると、
      * 「最後に終わったリクエスト」のセッション状態が残る
  * React Queryの `onMutate` でキャッシュを書き換えると、
      * **サーバ保存前に依存クエリのGETが飛ぶ** ことがある
  * そのGETが「古い状態」を保持したまま終了すると、
      * セッションが古い値で上書きされ、保存した値が消えたように見える

対策としては以下の通りです。

  * **フロントエンド：**
      * 「セッションに紐づく状態」を扱うmutationでは、安易に `onMutate` でQueryKeyに影響する値を変えない
      * 依存クエリは `enabled` などで発火条件を厳密にする
  * **バックエンド：**
      * 状態をセッションから専用テーブルに寄せ、DBのロックメカニズムで整合性を取る
      * 場合によっては Session Blocking で並列リクエスト自体を抑制する

「セッションの挙動」と「React Queryの挙動」をそれぞれ単体で理解していても、組み合わさった時にこういう事故が起きる、というのはなかなか気づきにくいポイントだと思います。

「保存したはずの値が、たまにだけ消える」「ログを見ると、保存APIの中ではちゃんと入っているのに…」という現象に悩まされている場合は、**セッション・並列リクエスト・楽観的更新の組み合わせ** を一度疑ってみてください。