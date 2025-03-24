---
title: WEBサイトにおけるRSSの仕組みとNext.js・Astro.jsでの実装
tags: [RSS]
createDate: 2025-03-24
updateDate: 2025-03-24
slug: rss-init
---

## はじめに

インターネット上の情報は長らく検索エンジンによって検索可能でしたが、段々と広告やサイトのクオリティの低下が目立つようになり、生成AIの出現とともに直接WEB上のサイトを閲覧する機会が減りつつあります。

一方で、信憑性の高い情報や興味深い仮説をブログで公開している方も多くいるため、効率的にそれらを収集したいという思いがあります。

そのための技術として、RSSとその関連技術について調査してみます。私がエンジニアになった時点でもはや廃れつつある技術だったんですが、個人的に特定の技術ブログを追いたいと感じることが多いです。

## RSSの基本的な仕組み

### RSSとは

まずRSSの名称ですが、バージョンによって名称が異なるようです。

[RSS - Wikipedia](https://ja.wikipedia.org/wiki/RSS)

- RSS 0.9x Rich Site Summary
- RSS 0.9,RSS 1.x系 RDF Site Summary
- RSS 2.0系 Really Simple Syndication

RSSの最大の利点は、チェックしたいサイトを個別に確認する必要がなく、複数サイトで更新されたページだけを一目で確認できることです。これにより、ユーザーは効率的に情報収集を行うことができます。

### RSSの技術的仕組み

RSSは、Webサイト上の更新情報や概要を配信するための文章上のルールを指します。技術的には、XMLという言語を使用して必要な情報（更新情報や概要）を構造化しています。

Webサイト側はRSSを受け取りたいユーザーへ向けて、Webサイトの更新情報や新規ページの概要をまとめた「RSSフィード」を配信します。ユーザー側はRSSリーダーというソフトウェアを使用して、XMLで記述されたRSSフィードから必要な情報だけを読み取ります。RSSリーダーを使うことで、各ページに設定されているRSSフィードを読み取り、ユーザーに更新情報が届くという仕組みです。

### RSSフィードの構造

RSSフィードは、XMLフォーマットで記述されています。基本的なRSSフィードの構造は以下のようになっています：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
  <channel>
    <title>サイトのタイトル</title>
    <link>サイトのURL</link>
    <description>サイトの説明</description>
    <language>ja</language>
    <pubDate>公開日時</pubDate>
    <item>
      <title>記事のタイトル</title>
      <link>記事のURL</link>
      <description>記事の概要</description>
      <pubDate>公開日時</pubDate>
    </item>
    <!-- 複数の<item>要素が続く -->
  </channel>
</rss>
```

この構造において、`<channel>` 要素はフィード全体の情報を含み、`<item>` 要素は個々の記事や更新情報を表します。各要素には、タイトル、リンク、説明、公開日時などの情報が含まれます。このXML形式のデータをRSSリーダーが解析し、ユーザーに分かりやすく表示することで、効率的な情報収集が可能になります。

### RSSとATOMの違い

ATOMもWebサイトの更新情報を知る手段の一つです。RSSと名前こそ異なっていますが、基本的な機能は同じものです。ATOMはRSSの機能に加えて、コンテンツを編集する機能も備えています。ATOMはRSSに改良を加えたものであり、ほぼ違いがないものという認識で問題ありません。

RSSは歴史的な経緯が長く、そのしがらみがない形でAtomが作成されたらしいですが、こちらはIETFの提案標準RFC4287として採択されています。

### RSSとRSSフィードの違い

RSSとRSSフィードは混同されがちですが、それぞれ違うものです。RSSは、Webサイトの更新情報を配信する「仕組み」のことで、RSSフィードはRSSに基づいた「データ（ファイル）」を指します。つまり、「仕組み」と、「その仕組みを利用したデータ」という違いと捉えることができます。

### RSSリーダーの種類と役割

RSSリーダーには、以下のような種類があります：

- メーラー型：メールソフトのようにWebサイトの更新情報を確認できる
- プラグイン型：ブラウザに組み込んで使用できる
- 通知型：更新情報をポップアップで通知してくれる
- ティッカー型：デスクトップに設置できる
- ホスティング型：特定のWebサイトのアカウントでRSSを利用する

代表的なRSSリーダーとしては、Feedly、Inoreader、Tiny Tiny RSS、The Old Reader、NewsBlur、Feederなどがあります。これらのRSSリーダーを使用することで、ユーザーは複数のWebサイトの更新情報を一箇所で効率的に確認することができます。

個人的にはずっとFeedlyを利用していました。

### RSSの利点と活用方法

RSSを活用することで、以下のようなメリットがあります：

効率的な情報収集が可能になります。複数のWebサイトの更新情報を一箇所で確認できるため、チェックしたいサイトを個別に訪問する必要がなくなります。これにより、情報収集にかかる時間を大幅に短縮することができます。

情報の整理も容易になります。関心のあるトピックごとにフォルダ分けして管理できるため、必要な情報だけを選別して閲覧することができます。また、更新されたコンテンツだけを確認できるため、情報収集の時間を短縮できます。さらに、自動で更新情報を取得するため、定期的なチェックが不要になるという利点もあります。

## RSSからAtomへの流れとその解説

### Atomの誕生背景

RSSは1999年に登場し、Webサイトの更新情報を配信する標準的な方法として広く普及しました。しかし、RSSには仕様の曖昧さや制限があり、それらの問題を解決するために2003年にAtom（正式名称：Atom Syndication Format）が開発されました。

Atomの開発は、RSSの複数のバージョン（RSS 0.9x、RSS 1.0、RSS 2.0）が存在し、それぞれに互換性の問題があったことが大きな要因でした。また、RSSの仕様が明確に定義されていない部分があり、実装によって解釈が異なるという問題もありました。

### RSSとAtomの主な違い

Atomは、RSSの問題点を解決するために設計されており、以下のような違いがあります：

1. **標準化レベル**：AtomはIETF（Internet Engineering Task Force）によってRFC 4287として標準化されており、明確な仕様が定義されています。一方、RSSは公式の標準化団体によって管理されていません。

2. **名前空間**：Atomは常にXML名前空間を使用しますが、RSSでは名前空間の使用が一貫していません。

3. **必須要素**：Atomでは、各エントリーに一意のIDが必須であり、更新日時（updated）と作成日時（published）を区別します。RSSでは、これらの要素が必須ではなく、日付も単一の要素（pubDate）しかありません。

4. **コンテンツモデル**：Atomはより柔軟なコンテンツモデルを持ち、プレーンテキスト、HTML、XHTML、バイナリなど様々な形式のコンテンツを明示的に指定できます。

5. **拡張性**：Atomは拡張性を考慮して設計されており、外部の名前空間を使用した拡張が容易です。

### Atomフィードの基本構造

Atomフィードの基本的な構造は以下のようになっています：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title>サイトのタイトル</title>
  <link href="https://example.com/"/>
  <updated>2025-03-24T10:00:00Z</updated>
  <author>
    <name>著者名</name>
  </author>
  <id>urn:uuid:60a76c80-d399-11d9-b93C-0003939e0af6</id>
  
  <entry>
    <title>記事のタイトル</title>
    <link href="https://example.com/article1"/>
    <id>urn:uuid:1225c695-cfb8-4ebb-aaaa-80da344efa6a</id>
    <updated>2025-03-23T18:30:02Z</updated>
    <summary>記事の概要</summary>
    <content type="html"><![CDATA[記事の本文（HTMLタグを含む）]]></content>
  </entry>
  <!-- 複数の<entry>要素が続く -->
</feed>
```

### 現在の状況と使い分け

現在では、RSSとAtomの両方が広く使用されています。多くのWebサイトやブログプラットフォームは、両方のフォーマットをサポートしています。例えば、WordPressは標準でRSS 2.0とAtomの両方のフィードを生成します。

どちらを選ぶべきかについては、以下のような基準があります：

- **互換性**：古いRSSリーダーとの互換性が重要な場合は、RSS 2.0が適しています。
- **国際化対応**：多言語コンテンツや特殊文字を扱う場合は、Atomの方が適しています。
- **標準化**：明確に定義された標準に準拠したいなら、Atomが適しています。
- **シンプルさ**：より単純な実装を望むなら、RSS 2.0が適しています。

多くの場合、両方のフォーマットを提供することで、より広範なユーザーに対応できます。

### 実装の観点から見たAtom

Atomを実装する際には、以下の点に注意が必要です：

1. **Content-Type**：Atomフィードを配信する際は、Content-Typeを`application/atom+xml`に設定します。

2. **必須要素**：Atomでは、`id`、`title`、`updated`が必須要素です。また、各エントリーにも同様に`id`、`title`、`updated`が必須です。

3. **ID生成**：Atomでは各エントリーに一意のIDが必須です。通常、URIベースのIDを使用します（例：`urn:uuid:...`や`https://example.com/articles/123`など）。

4. **日付フォーマット**：日付はISO 8601形式（例：`2025-03-24T10:00:00Z`）で指定する必要があります。

多くのフレームワークやライブラリは、RSSとAtomの両方のフォーマットをサポートしており、開発者は簡単に両方のフィードを生成できるようになっています。

## Next.jsでのRSS実装

### Next.jsとは

Next.jsは、Reactベースのフロントエンドフレームワークで、サーバーサイドレンダリング（SSR）や静的サイト生成（SSG）などの機能を提供しています。Next.jsは、Reactの開発体験を向上させるために設計されており、ルーティング、データフェッチング、画像最適化などの機能が組み込まれています。Next.jsでRSSフィードを実装する方法はいくつかありますが、主に「feedライブラリ」と「rssライブラリ」を使用する2つのアプローチが一般的です。

### feedライブラリを使用した実装

#### 1. ライブラリのインストール

まず、feedライブラリをインストールします：

```bash
# npmの場合
npm install feed

# yarnの場合
yarn add feed
```

#### 2. フィード情報を作成する関数

次に、RSSフィードを生成する関数を作成します。以下は`lib/feed.ts`などのファイルに実装する例です：

```typescript
// lib/feed.ts
import { Feed } from 'feed';

export const generateRssFeed = async (): Promise<string> => {
  // 環境変数などで設定した基本URLを取得
  const baseUrl = 'localhost:3000';
  
  // フィードの基本情報を設定
  const feed = new Feed({
    title: 'サイトのタイトル',
    description: 'サイトの説明',
    id: baseUrl,
    link: baseUrl,
    language: 'ja',
    copyright: 'copyright',
    generator: baseUrl,
  });

  // データを取得する（APIやファイルなどから）
  const posts = await getPosts();
  
  // 各記事をフィードに追加
  posts.forEach((post) => {
    feed.addItem({
      title: post.title,
      description: post.description,
      date: new Date(post.date),
      id: post.url,
      link: post.url,
    });
  });

  // RSS 2.0形式でフィードを出力
  return feed.rss2();
  
  // 他の形式でも出力可能
  // feed.atom1() でAtomフィードを生成
  // feed.json1() でJSONフィードを生成
};
```

#### 3. XMLファイルを生成する

Next.jsでは、Pages RouterとApp Routerの2つのルーティングシステムがあり、それぞれで実装方法が異なります。

##### Pages Routerの場合（pages/feed.tsx）

```typescript
// pages/feed.tsx
import { GetServerSideProps } from 'next';
import { generateRssFeed } from '../lib/feed';

export const getServerSideProps: GetServerSideProps = async ({ res }) => {
  const xml = await generateRssFeed();
  
  res.statusCode = 200;
  res.setHeader('Cache-Control', 's-maxage=86400, stale-while-revalidate');
  res.setHeader('Content-Type', 'text/xml');
  res.end(xml);

  return {
    props: {},
  };
};

const Page = (): null => null;
export default Page;
```

##### App Routerの場合（app/rss.xml/route.ts）

```typescript
// app/rss.xml/route.ts
import { generateRssFeed } from '@/lib/feed';

export const revalidate = 60 * 60 * 24; // 24時間ごとに再検証

export async function GET() {
  const xml = await generateRssFeed();
  
  return new Response(xml, {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': `s-maxage=${revalidate}, stale-while-revalidate`
    }
  });
}
```

### rssライブラリを使用した実装

feedライブラリの代わりに、rssライブラリを使用することもできます。

#### 1. ライブラリのインストール

```bash
# npmの場合
npm install rss

# yarnの場合
yarn add rss
```

#### 2. App Routerでの実装例

```typescript
// app/rss.xml/route.ts
import { description, title } from '@/scripts/metadata'; 
import { getArticles } from '@/scripts/microcms'; 
import Rss from 'rss';

const url = String(process.env.NEXT_PUBLIC_BASE_URL); 

export const revalidate = 60 * 60 * 24 * 1; // 1日ごとに再検証

export async function GET() {
  const feed = new Rss({
    title,
    description,
    feed_url: `${url}/rss.xml`,
    site_url: url,
    language: 'ja'
  });

  const { contents } = await getArticles();
  
  for (let i = 0; i < contents.length; ++i) {
    feed.item({
      title: contents[i].title,
      description: `${contents[i].description.description}`, 
      url: `${url}/${contents[i].url}`,
      date: contents[i].updatedAt
    });
  }

  return new Response(feed.xml(), {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': `s-maxage=${revalidate}, stale-while-revalidate`
    }
  });
}
```

### 実装時の注意点

Next.jsでRSSフィードを実装する際には、いくつかの注意点があります。

キャッシュ設定に関しては、RSSフィードは頻繁に更新されるわけではないため、適切なキャッシュ設定を行うことでサーバーの負荷を軽減できます。`Cache-Control`ヘッダーを使用して、キャッシュの有効期間を設定しましょう。

コンテンツのエスケープも重要です。XMLでは特殊文字（`<`, `>`, `&`など）をエスケープする必要があります。多くのRSSライブラリは自動的にエスケープを行いますが、HTMLコンテンツを含める場合は`CDATA`セクションで囲むことを検討してください。

パフォーマンスについては、大量の記事を含むRSSフィードは生成に時間がかかる場合があります。

また、RSSフィードをGoogle Search Consoleに登録することで、検索エンジンがコンテンツを効率的にクロールできるようになります。これにより、サイトのSEO対策にも貢献します。

## Astro.jsでの実装

自分がブログを作成する際に利用しているAstro.jsでも実装してみます。

### パッケージのインストール

まずは公式で用意されているRSS統合パッケージをインストールします。

```bash
npm install @astrojs/rss
```

### RSSフィード生成ページを作成

Astro.jsでは、`src/pages/rss.xml.ts`というようなページを作成することでRSSフィードを生成できます。

自分のブログで実装する場合はこのような形になりそうです。

blogコレクションにあること、draft記事を除くことなどを指定しています。
サイトのタイトル等は`consts`ファイルで定義しています。

```typescript
import rss from '@astrojs/rss';
import { getCollection } from 'astro:content';
import { SITE_TITLE, SITE_DESCRIPTION, SITE_URL } from '../consts';
import type { APIContext } from 'astro';

export async function GET(context: APIContext) {
  const blog = await getCollection('blog');
  const posts = blog.filter((post) => !post.data.draft);

  return rss({
    title: SITE_TITLE,
    description: SITE_DESCRIPTION,
    site: context.site?.toString() || SITE_URL,
    items: posts.map((post) => ({
      title: post.data.title,
      pubDate: post.data.createDate,
      description: post.body.slice(0, 200) + '...',
      link: `/posts/${post.slug}/`,
      categories: post.data.tags,
    })),
    customData: `<language>ja</language>`,
  });
}
```

Astro.jsの場合はこれだけで`/rss.xml`にアクセスするとRSSフィードが生成されていることを確認できます。

SSGで作成しているブログなので、自分の場合は記事を追加するたびに簡単なCI/CDを利用してデプロイしているため、その際のビルドでRSSフィードも作成されます。

## まとめ

RSSは1999年に登場した比較的古い技術ですが、現在でも多くのWebサイトで利用されており、ユーザーが効率的に情報を収集するための強力なツールとなっています。RSSを実装することで、Webサイト運営者はユーザーに対して効率的な情報提供を行うことができます。

現在は個人開発でRSSリーダーを実装しようとしていたので、自分のブログのフィード配信と合わせて勉強になりました。

生成AIによる検索に頼る時代も到来するかもしれませんが、RSSはまだ重要な役割を果たすと思います。

## 参考資料

- [LaravelでRSS/Atomフィードを配信・生成する - Zenn](https://zenn.dev/ichii731/articles/84a62b88a17a7c)
- [Next.jsでfeedライブラリを使いRSSフィードを生成する - Qiita](https://qiita.com/kaeru333/items/3685f9231c9c07edb0e4)
- [Next.js（App Router）でRSSフィードを実装する - みかづきブログ・カスタム](https://blog.kimizuka.org/entry/2024/03/15/123320)
- [RSSとは？基本的な仕組みと使い方を解説 | マーケトランク](https://www.profuture.co.jp/mk/column/what-is-rss)
