# AGENTS.md — ziyun.rocks

## Project overview

This repository is the source for the personal blog at https://blog.ziyun.rocks. It is a Hugo static site deployed to Netlify on every push to `main`. The theme ("shimong") is implemented entirely as project-level `layouts/` and `static/` files — there is no `themes/` directory.

---

## Setup & commands

**Local preview (including drafts):**
```
hugo server -D
```

**Production-style build (matches Netlify exactly):**
```
hugo --gc --minify -b $URL
```

**Required Hugo version:** `0.162.1` (extended). Keep your local version in sync with `netlify.toml`'s `HUGO_VERSION`. Mismatches can cause template or deprecation differences between local builds and production.

---

## Project layout

```
config.toml          — site config: baseURL, title, taxonomies, menus, markup
netlify.toml         — build command, HUGO_VERSION, deploy contexts
content/
  about.md           — /about/ page
  posts/             — all blog posts (see Content authoring below)
    _index.md        — section metadata (title: "Archive")
layouts/
  _default/          — baseof.html, list.html, single.html, terms.html
  partials/          — head.html, seo.html, nav.html, sidebar.html, footer.html
  404.html
  index.html
static/
  css/shimong.css    — all site styles
  favicon.svg
  uploads/           — miscellaneous uploaded assets
docs/
  superpowers/
    specs/           — design documents (e.g. shimong theme migration spec)
    plans/           — implementation plans
  theme.md           — theme documentation (may be created concurrently; reference this path)
archetypes/          — Hugo content archetypes
```

---

## Content authoring

**Single-file post:**
```
content/posts/YYYY-MM-DD-slug-here.md
```

**Page bundle (post with colocated images):**
```
content/posts/YYYY-MM-DD-slug-here/
  index.md
  image.png
```

**Front matter (YAML):**
```yaml
---
title: "Post Title"
date: 2026-01-15
slug: optional-url-override   # omit to derive from filename
draft: true                   # omit when ready to publish
tags:                         # optional; taxonomy is enabled but rarely used
  - example
---
```

Post URLs are `/posts/<slug>/`. The `slug` front matter field overrides the filename-derived slug when present.

Use `<!--more-->` as the summary cut if you want an explicit excerpt on the list page.

---

## Conventions & gotchas

- **Never commit `public/`** — it is gitignored and is the Netlify build output.
- **Hugo version parity** — always match your local Hugo version to `HUGO_VERSION = "0.162.1"` in `netlify.toml`. The recent Bear→shimong migration fixed `languageCode`/`.Site.LanguageCode` deprecations for modern Hugo. Fix, do not suppress, any new Hugo deprecation warnings.
- **Theme lives in `layouts/` + `static/css/shimong.css`** — there is no `themes/` directory. All template and style changes go in those two places. Refer to `docs/theme.md` for theme design rationale.
- **Comments are utterances** (GitHub Issues-backed), injected only on `section == "posts"` via `layouts/_default/single.html`. The about page intentionally has no comments.
- **Preserve post URLs** — existing posts have inbound links. Do not rename slugs without a redirect.
- **Easter eggs in `layouts/partials/head.html`:**
  - `X-Clacks-Overhead: GNU Terry Pratchett` meta tag
  - `<link rel="me">` pointing to the Mastodon profile at `fedi.ziyun.rocks/@ziyun`

---

## Deployment

Pushing to `main` triggers a Netlify production build:
```
hugo --gc --minify -b $URL
```
with `HUGO_ENV=production`.

Additional Netlify deploy contexts (configured in `netlify.toml`):
- **deploy-preview** — runs `hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL` (includes future-dated posts)
- **branch-deploy** — runs `hugo --gc --minify -b $DEPLOY_PRIME_URL`

The Netlify Hugo cache is managed by `netlify-plugin-hugo-cache-resources`.
