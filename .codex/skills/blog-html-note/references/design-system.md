# Lyricrime-Inspired HTML Article Design System

Base the article design on the public `lyricrime.com` blog tone: minimal, text-first, white background, black text, simple navigation/list rhythm, and restrained decoration.

## Principles

- Prioritize reading. The design should feel like a quiet technical blog, not a landing page.
- Use a white or near-white page background.
- Use black or near-black text.
- Keep accents sparse and functional.
- Prefer simple borders and whitespace over colored panels.
- Avoid gradients, decorative blobs, heavy shadows, oversized hero sections, and card-heavy layouts.
- Use one-column article layout with diagrams that fit the content width.

## Layout

Use a centered article container:

```css
.page {
  max-width: 880px;
  margin: 0 auto;
  padding: 48px 20px 72px;
}
```

For narrow viewports:

```css
@media (max-width: 640px) {
  .page {
    padding: 28px 16px 56px;
  }
}
```

## Typography

Use system fonts.

```css
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  color: #111;
  background: #fff;
  line-height: 1.85;
}
```

Scale:

- Title: `clamp(2rem, 6vw, 4.8rem)` is allowed for the top page title only.
- Article body: `1rem` to `1.05rem`.
- Section headings: restrained, `1.25rem` to `1.7rem`.
- Diagram labels: `0.85rem` to `0.95rem`.

Do not use negative letter spacing.

## Color Tokens

```css
:root {
  --bg: #fff;
  --text: #111;
  --muted: #666;
  --line: #dedede;
  --soft: #f7f7f7;
  --accent: #111;
  --accent-soft: #efefef;
}
```

Use accent colors only when they clarify the diagram. If a second accent is needed, use a muted blue or green sparingly:

```css
--blue: #315f8c;
--green: #3f6f57;
```

## Components

### Header

Use date and tags as small muted text. Keep the title prominent but plain.

```html
<header class="article-header">
  <p class="meta">2026/06/14 · AI / Codex</p>
  <h1>記事タイトル</h1>
  <p class="lead">短い導入文。</p>
</header>
```

### Diagram

Diagrams should use borders, whitespace, and simple arrows.

```html
<section class="diagram" aria-label="記事化フローの図解">
  <div class="flow">
    <div class="node">User</div>
    <div class="arrow">→</div>
    <div class="node">Codex</div>
    <div class="arrow">→</div>
    <div class="node">HTML Article</div>
  </div>
</section>
```

```css
.diagram {
  margin: 32px 0;
  padding: 20px;
  border: 1px solid var(--line);
  background: var(--soft);
}

.flow {
  display: grid;
  grid-template-columns: 1fr auto 1fr auto 1fr;
  gap: 12px;
  align-items: center;
}

.node {
  min-height: 72px;
  padding: 14px;
  border: 1px solid var(--line);
  background: var(--bg);
  display: grid;
  place-items: center;
  text-align: center;
}

.arrow {
  color: var(--muted);
}

@media (max-width: 640px) {
  .flow {
    grid-template-columns: 1fr;
  }

  .arrow {
    transform: rotate(90deg);
    justify-self: center;
  }
}
```

### Checklist

Use simple bordered lists.

```css
.checklist {
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
  padding: 16px 0;
}
```

## Full HTML Skeleton

```html
<!--
title: "記事タイトル"
tags: ["AI", "Codex"]
createDate: 2026-06-14
updateDate: 2026-06-14
slug: 2026-06-14-example
draft: false
format: html
-->
<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>記事タイトル</title>
  <style>
    :root {
      --bg: #fff;
      --text: #111;
      --muted: #666;
      --line: #dedede;
      --soft: #f7f7f7;
      --accent: #111;
      --accent-soft: #efefef;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      color: var(--text);
      background: var(--bg);
      line-height: 1.85;
    }

    .page {
      max-width: 880px;
      margin: 0 auto;
      padding: 48px 20px 72px;
    }

    .meta {
      color: var(--muted);
      font-size: 0.92rem;
    }

    h1 {
      margin: 0 0 16px;
      font-size: clamp(2rem, 6vw, 4.8rem);
      line-height: 1.08;
      font-weight: 700;
      letter-spacing: 0;
    }

    h2 {
      margin-top: 44px;
      padding-top: 24px;
      border-top: 1px solid var(--line);
      font-size: 1.45rem;
      letter-spacing: 0;
    }

    .lead {
      font-size: 1.08rem;
      color: #333;
    }

    a {
      color: var(--text);
      text-underline-offset: 0.2em;
    }

    code {
      padding: 0.1em 0.3em;
      background: var(--accent-soft);
      font-size: 0.92em;
    }

    .diagram {
      margin: 32px 0;
      padding: 20px;
      border: 1px solid var(--line);
      background: var(--soft);
    }

    @media (max-width: 640px) {
      .page {
        padding: 28px 16px 56px;
      }
    }
  </style>
</head>
<body>
  <main class="page">
    <header class="article-header">
      <p class="meta">2026/06/14 · AI / Codex</p>
      <h1>記事タイトル</h1>
      <p class="lead">短い導入文。</p>
    </header>

    <section>
      <h2>見出し</h2>
      <p>本文。</p>
    </section>
  </main>
</body>
</html>
```
