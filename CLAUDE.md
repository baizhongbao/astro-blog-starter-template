# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Astro blog starter template deployed as a static site on Cloudflare Workers. Uses Astro's content collections for Markdown/MDX blog posts with Zod schema validation.

## Commands

- `npm run dev` — Local dev server at `localhost:4321`
- `npm run build` — Production build to `./dist/`
- `npm run preview` — Build + preview via Wrangler
- `npm run deploy` — Deploy to Cloudflare Workers
- `npm run check` — Full validation: build + TypeScript check + dry-run deploy
- `npm run cf-typegen` — Regenerate Cloudflare Worker types after `wrangler.json` changes

## Architecture

- **Static-first**: Pages are pre-rendered at build time. The Cloudflare adapter serves the built output as a Worker.
- **Content Collections**: Blog posts live in `src/content/blog/` as Markdown/MDX. Frontmatter is validated by a Zod schema (defines `title`, `description`, `pubDate`, `heroImage`).
- **File-based routing**: `src/pages/` maps directly to URL routes. Blog detail uses `[...slug].astro` dynamic route with `getStaticPaths`.
- **Site config**: Site title and description are in `src/consts.ts` (imported as needed, not in astro.config).
- **Integrations**: MDX (`@astrojs/mdx`), sitemap (`@astrojs/sitemap`), RSS (`@astrojs/rss`).

## Adding Blog Posts

Create a `.md` or `.mdx` file in `src/content/blog/` with this frontmatter:

```yaml
---
title: "Post title"
description: "Post description"
pubDate: "Mon DD YYYY"
heroImage: "/path-to-image.jpg"
---
```

## Key Files

- `astro.config.mjs` — Astro config (integrations, Cloudflare adapter, site URL)
- `wrangler.json` — Cloudflare Worker config
- `src/content/config.ts` — Content collection schema (create if adding new collections)
- `src/consts.ts` — Global site metadata
