---
name: outreachbox-content-engine
description: Research a topic and turn it into articles, social posts, a monthly content calendar, or brand designs in OutreachBox. Use when the user wants to generate marketing content, plan a content calendar, research what to write about, create social posts, or publish to a connected social account.
---

# Content engine

Research, generate, schedule, publish. Each stage stands alone, but output quality drops sharply if you skip research and generate from a bare prompt.

## 0. Project context

Nearly everything here takes a `projectId`. `projects_list`, or `projects_create` with the user's real site URL — it gets crawled to build brand context, which is what keeps generated copy sounding like the user rather than generic AI.

`projects_get` returns that context. Read it before writing anything substantial.

## 1. Research the topic

`research_topic_create` is asynchronous and needs all of:

```json
{
  "name": "Payments infrastructure Q4",
  "keywords": ["embedded payments", "payment orchestration"],
  "researchTypes": ["News Articles", "Industry Trends"],
  "geographicTarget": "United Kingdom",
  "timeRange": "90",
  "depthLevel": 3,
  "projectId": "..."
}
```

`timeRange` is days and must be one of `"7"`, `"30"`, `"90"`, `"365"` — a string, not a number. `depthLevel` is 1–5; 3 is a sensible default, 5 is slow and only worth it for a cornerstone piece.

Poll `research_topic_get` for results.

## 2. Generate

`content_generate`. Only `contentType` is required (`social_post`, `article`, `email_template`).

```json
{
  "contentType": "article",
  "title": "What embedded payments actually cost",
  "prompt": "Write for CTOs evaluating build vs buy. Ground it in the Q4 research.",
  "keywords": ["embedded payments", "payment orchestration"],
  "tone": "direct, practical",
  "length": "long",
  "projectId": "...",
  "enableSERP": true,
  "humanizeContent": true
}
```

`enableSERP: true` pulls live search facts in — use it for anything factual or time-sensitive, skip it for evergreen opinion. For social, set `platform` (`linkedin`, `twitter`) so length and format fit.

Retrieve with `content_list` / `content_get`. Always show the user the draft before publishing. Generated content is a first draft.

## 3. Plan a calendar

`content_calendar_generate` builds a month at a time:

```json
{ "projectId": "...", "month": 9, "year": 2026, "postsPerMonth": 12,
  "keywords": ["embedded payments"], "enableSERP": true }
```

**`month` is 0-indexed** — 0 is January, 9 is October. This is the single easiest thing to get wrong here; confirm the month you meant.

Leave `autoPublish` off unless the user explicitly asks for it. It publishes without further review.

Inspect with `content_calendar_list` and `content_calendar_get`.

## 4. Designs

`designs_generate` needs `projectId` and `keywords`; `platforms`, `topic` and `count` shape the output. It produces on-brand visuals to pair with posts.

## 5. Publish

`social_accounts_list` for connected accounts and their ids. Nothing connected means the user must link an account in the web app.

`social_publish` takes `accountId` plus either a saved `contentId` or raw `content`.

**Confirm before publishing.** This posts publicly under the user's brand and cannot be quietly undone. Show the exact text and the destination account, and wait for a yes.

## Order that works

Research → generate → review with the user → publish. When the user wants volume, build the calendar first, then generate against its slots. Generating in bulk without research produces content that reads like everyone else's.
