---
name: blog-html-note
description: Create, commit, and push publishable Japanese HTML blog-note articles in this blog_article project. Use when the user asks to prepare project blog skills, explicitly invokes this skill, asks to turn a question into an article, starts the request with /blog, or confirms after an answer that the content should become an article. Produces HTML articles with simple explanatory diagrams, Lyricrime-inspired minimal styling, privacy checks, a local git commit, and a push to the current branch.
---

# Blog HTML Note

## Core Rule

Create an article only in either case:

- The user input starts with `/blog`.
- After answering a normal question, ask whether to make it an article and the user clearly confirms.

For normal questions that do not start with `/blog`, answer the question first. Do not create files or commits until the user confirms article creation.

## Output Contract

Create public-ready HTML articles under:

```text
Article/note/YYYY/MM/YYYY-MM-DD-slug.html
```

Use HTML, not Markdown. The file copied to `astro-blog` is hosted as HTML.

Include YAML-like metadata in an HTML comment at the top so sync scripts can parse or preserve publishing data:

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
```

Use `draft: false`. In this workflow, PR merge means publication approval.

## Workflow

1. Determine whether the request is article-eligible.
2. If article-eligible, inspect existing `Article/note/` naming if needed.
3. Create a concise Japanese article as a complete HTML document.
4. Include at least one HTML/CSS diagram that explains the core idea visually.
5. Use the Lyricrime-inspired design system in `references/design-system.md`.
6. Run privacy checks before writing or committing.
7. Add only the article and necessary assets.
8. Commit the article with a focused message.
9. Push the commit to the current branch.

Commit message format:

```text
Add note: <short article title>
```

If the worktree has unrelated user changes, leave them untouched and commit only files created or changed for the article.

Push rule:

- If the current branch already tracks a remote branch, run `git push`.
- If there is no upstream branch, run `git push -u origin <current-branch>`.
- Do not include unrelated local changes in the commit before pushing.

## Article Structure

Use this order:

1. Metadata comment
2. `<!doctype html>` document
3. `<head>` with responsive CSS
4. Header with date, title, and short lead
5. Main article body
6. One or more visual explanation sections
7. Practical checklist or implementation steps
8. Short conclusion

Prefer plain, practical Japanese. Avoid marketing copy.

## Diagram Requirements

Every article must include at least one diagram made with HTML and CSS. Use semantic HTML where possible.

Good diagram patterns:

- Flow diagram for workflows
- Comparison grid for tradeoffs
- Layer diagram for architecture
- Timeline for step-by-step processes
- Responsibility map for system boundaries

Do not rely on external CDNs, remote fonts, or JavaScript unless the user specifically asks.

## Privacy Rules

Never include:

- company-internal information
- internal URLs
- private repository names
- customer names
- personal names from conversations
- Slack or internal chat messages
- credentials, tokens, API keys
- AWS account IDs or internal identifiers
- salary, loan, family, health, address, or other private life details
- security review findings
- unreleased product details

Generalize company-specific context:

```text
NG: 特定企業の予約管理でFeature Flagを導入する場合
OK: ある予約管理システムでFeature Flagを導入する場合
```

## Quality Rules

- Write in Japanese.
- Keep the article useful without being long for its own sake.
- Mark assumptions clearly.
- Do not fabricate facts, links, benchmarks, or references.
- Browse or verify when current facts, product behavior, or external references matter.
- Prefer public primary sources when references are necessary.
- Keep text readable on mobile.
- Ensure diagram text does not overlap at narrow widths.

## Pull Request Notes

When asked to create a PR, include:

```md
## Summary

- Added HTML note article: Article/note/YYYY/MM/YYYY-MM-DD-slug.html

## Privacy checklist

- [ ] Input started with `/blog` or user confirmed article creation
- [ ] `draft: false`
- [ ] No company-internal information
- [ ] No internal URLs
- [ ] No private repository names
- [ ] No customer or personal names
- [ ] No credentials, tokens, or account IDs
- [ ] No private life details
- [ ] No security review findings
- [ ] No unreleased product details
- [ ] Content is generalized enough for public publication

## Verification

- [ ] Checked HTML structure
- [ ] Checked responsive diagram layout
- [ ] Checked local diff
```

## References

Read `references/design-system.md` before creating or updating article HTML.
