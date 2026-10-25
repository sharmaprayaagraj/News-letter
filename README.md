<div align="center">

# The Daily Ledger

**A print-style newspaper front end built on the Next.js App Router, streaming live headlines from Nigerian, Indian and global wires.**

[![CI](https://github.com/CoderGamerUnknow/News-letter/actions/workflows/ci.yml/badge.svg)](https://github.com/CoderGamerUnknow/News-letter/actions/workflows/ci.yml)
[![Next.js](https://img.shields.io/badge/next.js-16.4-000000?style=flat-square&logo=next.js)](https://nextjs.org)
[![React](https://img.shields.io/badge/react-19.3-61dafb?style=flat-square&logo=react)](https://react.dev)
[![License: MIT](https://img.shields.io/badge/license-MIT-3b82f6?style=flat-square)](LICENSE)

<img src="public/social-cover.png" alt="The Daily Ledger — masthead and front-page columns" width="100%">

</div>

## Overview

A complete newspaper generated from a single typed dataset — no CMS, no database. It
builds to static HTML you can host anywhere, and it pulls real headlines from global
news wires at build time, refreshing them in the browser afterwards without a rebuild.

The layout is deliberately print-like: a masthead, column rules and hairline dividers,
serif display type against a measured body face, and front-page hierarchy that changes
between lead, secondary and latest stories.

## Highlights

- **Content at scale** — 20 hand-written features plus a seeded, deterministic wire-story
  generator (`lib/generate.ts`) that fills the edition to hundreds of stories. Bump
  `WIRE_STORY_COUNT` in `lib/articles.ts` to grow it.
- **Real news wire** — 36 RSS feeds across three regions are pulled at build time and shown
  in the breaking ticker, per-section "Live Wire" blocks and the region browser. Every
  region × sub-topic has at least two sources, a shared headline blocklist filters junk,
  and a feed that is down, slow or malformed is skipped rather than failing the build.
- **News by region** — `/news` groups live headlines into **Nigerian**, **Indian** and
  **Global** news, each listing its sub-topics as always-visible headings. The region tabs
  are a full ARIA tablist with arrow-key navigation.
- **Live updates** — the ticker and live wires poll `/api/live` every few minutes. The
  route caches the wire for three minutes and returns the newest headlines per region.
- **Instant routes** — article, section and author pages are fully prerendered;
  `dynamicParams = false` turns unknown slugs away with a true 404 from the first byte
  (no soft-404).
- **Search** — `/search` filters the whole edition in the browser.
- **File news by hand** — the password-gated `/admin` desk lets editors write a story,
  preview it, confirm, and publish it into the edition (front page, sections, search,
  feed, sitemap) without touching code.
- **Related stories** — scored by keyword overlap, not just section.
- **Real photos** — each card layers a seeded photo from `picsum.photos` over procedural
  SVG art, which acts as the offline fallback.
- **Social cards** — a generated OpenGraph/Twitter image per article.
- **Share + reactions** — Web Share / copy-link and local-only reaction marks.
- **Pagination** — section grids reveal a page at a time with "Load more".

## Tech stack

| Layer | Choice |
| --- | --- |
| Framework | Next.js 16.4 (App Router, Turbopack) |
| UI | React 19.3 |
| Language | TypeScript 5 |
| Styling | CSS Modules + design-token variables |
| Testing | Vitest |
| Linting | ESLint (`next/core-web-vitals` + TypeScript) |
| Deploy | Any Node host — Netlify adapter included |

## Getting started

```bash
git clone https://github.com/CoderGamerUnknow/News-letter.git
cd News-letter
npm install

npm run dev      # http://localhost:3000
npm run build    # fetch live news, then type-check + production build
npm start        # serve the production build
```

### Scripts

| Script | What it does |
| --- | --- |
| `npm run dev` | Next dev server |
| `npm run build` | `prebuild` fetches the wire snapshot, then `next build` |
| `npm start` | Serve the production build |
| `npm run news` | Refresh `lib/live-news.json` on its own |
| `npm run lint` | ESLint |
| `npm test` | Vitest |

`npm run build` runs `scripts/fetch-news.mjs` as a `prebuild` step, pulling headlines from
the RSS registry into `lib/live-news.json`. If the network is down it keeps the previous
snapshot and never fails the build.

## Admin desk

Editors can file stories by hand at **`/admin`** — no code changes required.

1. Set a shared password and restart:

   ```bash
   ADMIN_PASSWORD=your-secret npm run dev    # or .env.local for local dev
   ```

2. Open `/admin`, sign in, and use **Write a story**. The form takes headline, dek,
   section, byline, date, slug (auto-derived from the headline) and body, with one
   paragraph per blank line.
3. **Preview story** renders the article exactly as its page will read, using the same
   validation rules the server enforces.
4. **Confirm & publish** writes it to `lib/manual-news.json`. The story then joins the
   edition everywhere — front page, section fronts, search, related stories, `feed.xml`
   and `sitemap.xml` — and `/admin` lists filed stories for editing or withdrawing.

Published stories are build-time data: on a deployed site, commit the updated
`lib/manual-news.json` (or run `npm run build`) to take it live. The store is a plain
committed JSON file, so this works on any host that can write the repo; serverless
filesystems are read-only, so run the desk locally or on a Node host. The password guards
both the page and `/api/admin/stories`; sessions are signed cookies that die when
`ADMIN_PASSWORD` rotates.

## Routes

| Route | Source | Notes |
| --- | --- | --- |
| `/` | `app/page.tsx` | Lead story, secondary column, latest grid, live wires |
| `/article/[slug]` | `app/article/[slug]/page.tsx` | Full story, share row, reactions, related stories |
| `/article/[slug]/opengraph-image` | `app/article/[slug]/opengraph-image.tsx` | Generated social card per story |
| `/section/[slug]` | `app/section/[slug]/page.tsx` | Section front, paginated grid, live wire |
| `/author/[slug]` | `app/author/[slug]/page.tsx` | Every story by one contributor |
| `/news` | `app/news/page.tsx` | Live headlines by region (Nigerian / Indian / Global) |
| `/search` | `app/search/page.tsx` | Client-side search across the edition |
| `/admin` | `app/admin/page.tsx` | Password-gated desk for hand-filing stories |
| `/api/live` | `app/api/live/route.ts` | On-demand headlines for client polling |
| `/api/admin/login`, `/api/admin/logout` | `app/api/admin/*` | Shared-password sign-in and sign-out |
| `/api/admin/stories` | `app/api/admin/stories/route.ts` | Publish, update and withdraw hand-filed stories |
| `not-found` | `app/not-found.tsx` | Unknown slugs and unmatched URLs |
| `/sitemap.xml`, `/robots.txt`, `/feed.xml` | `app/sitemap.ts`, `app/robots.ts`, `app/feed.xml/route.ts` | Generated from the same dataset |

## Project structure

```
app/                      App Router pages, routes and components
  article/                Article page + generated OpenGraph card
  section/                Section front with paginated grid
  author/                 Contributor archive
  news/                   Region browser
  search/                 Client-side search
  admin/                  Password-gated desk
  api/live/               On-demand headline endpoint
  api/admin/              Sign-in, publish, update and withdraw endpoints
  components/             Shared UI (cards, ticker, live wire, art, admin)
lib/
  articles.ts             Curated features + generated wire stories
  generate.ts             Seeded, deterministic story generator
  manual-news.json        Stories filed by hand through /admin
  manual-news.ts          Validation rules shared by form and API
  manual-store.ts         Filesystem persistence for hand-filed stories
  admin-auth.ts           Shared-password sessions for the admin desk
  feeds.json              Region + RSS feed registry (single source of truth)
  headline-blocklist.json Shared junk-headline filter
  rss.ts                  Dependency-free runtime RSS parser
  live-news.ts            Typed loader over the build-time snapshot
  search.ts               Browser-side edition search
scripts/
  fetch-news.mjs          Build-time wire fetch
  make-social-card.mjs    Regenerates public/social-cover.png
netlify.toml              Netlify build settings
```

## Configuration

Set `NEXT_PUBLIC_SITE_URL` to your real origin so metadata, the sitemap and the feed use it
instead of the placeholder `siteUrl`:

```bash
NEXT_PUBLIC_SITE_URL=https://your-domain.com npm run build
```

The only build-time network dependency is the news fetch, and it degrades gracefully to the
last snapshot. `/api/live` is dynamic and needs a Node runtime, so it is not part of a pure
static export.

### Netlify

Netlify's OpenNext adapter (`@netlify/plugin-nextjs`) runs the Next.js app; `netlify.toml`
pins the build command, Node version and the adapter, so the deploy works even if the site
was created without the framework preset. Don't add a `publish` directory — publishing the
raw `.next` folder serves build internals and 404s every path.

The adapter cannot package Next 16's Node-runtime `proxy.ts` (the renamed middleware) — the
deploy fails while bundling it as an Edge Function. This project therefore keeps slug
validation in the page and ships no middleware, so an unknown dynamic slug still lands on
the 404 page.

## CI

`.github/workflows/ci.yml` runs on every push and pull request to `main`, cancelling
superseded runs: `npm ci` → lint → `next typegen` → `tsc --noEmit` → `npm test` →
`npm run build`. Route types have to be generated first because a fresh clone has no
`.next/types`, and `next typegen` writes them without a production build.

## Contributing

Fork the repository, create a feature branch and open a pull request. `npm run lint`,
`npm test` and `npm run build` should all pass before submitting.

## License

[MIT](LICENSE) © 2026 CoderGamerUnknow
