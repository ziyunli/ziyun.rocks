# Shimong Theme Reference

## Overview

The blog uses a custom adaptation of the **shimong** theme (originally a Zola theme at
https://github.com/ziyunli/zola-shimong). It is implemented entirely as project-level
`layouts/` and `static/` — there is no `themes/` directory and no theme dependency in
`config.toml`.

Signature look: a giant vertical CJK site title (子雲) in a left sidebar, monospace
typography throughout, a chronological post archive on the homepage, `blog · about · tags`
nav with `·` separators, monokai dark code blocks, and automatic dark mode via
`prefers-color-scheme`.

Deployed via Netlify (Hugo 0.162.1 extended) to https://blog.ziyun.rocks.

---

## Page Structure

Every page uses the same shell from `layouts/_default/baseof.html`. The `<body>` is a
flex row: a `<header>` sidebar on the left and a `#main` content column on the right.

```
<body>                        (display:flex; flex-flow:row)
  <header>                    left sidebar — giant 子雲 title, links home
  <div id="main">             content column, max-width 820px, flex column
    <nav>                     blog · about · tags  (centered)
    <main>                    page-specific content (the "main" block)
    <footer>                  LLM disclaimer
  </div>
</body>
```

The `<header>` holds only `partials/sidebar.html`. The `#main` div holds `partials/nav.html`,
the `{{ block "main" }}` slot, and `partials/footer.html`.

---

## Templates

| File | Rendered for | Notes |
|---|---|---|
| `layouts/_default/baseof.html` | All pages | HTML shell; wires partials + `main` block |
| `layouts/index.html` | Homepage (`/`) | Archive list of all `posts` section pages, date + title |
| `layouts/_default/single.html` | Any single content page | Title, date, tags (via `.GetTerms "tags"`), content; utterances comments footer gated on `eq .Section "posts"` |
| `layouts/_default/list.html` | Section/term listings (`/posts/`, `/tags/<tag>/`) | Section title + article-list of child pages |
| `layouts/_default/terms.html` | Taxonomy index (`/tags/`) | Alphabetical list of tags with counts |
| `layouts/404.html` | 404 error page | Plain "page wandered off" message with home link |

### single.html details

- Date renders as `2006-01-02` (hardcoded, ignoring `params.dateFormat`).
- Tags render inline after an `&mdash;` as `#tagname` links.
- The utterances comments block (`<script src="https://utteranc.es/client.js">`) is
  wrapped in `{{ if eq .Section "posts" }}` so it only appears on posts, not on
  standalone pages like `/about/`.
- utterances config: `repo="ziyunli/ziyun.rocks"`, `issue-term="pathname"`,
  `label="comments"`, `theme="preferred-color-scheme"`.

### index.html and list.html dates

Both use `{{ .Date.Format (default "2006-01-02" .Site.Params.dateFormat) }}`, so the
`params.dateFormat` config value is respected here.

---

## Partials

### `layouts/partials/head.html`

- `X-Clacks-Overhead: GNU Terry Pratchett` HTTP-equiv meta tag.
- `<meta charset="utf-8">` and viewport tag.
- `<title>`: site title on home, `Page Title | Site Title` elsewhere.
- `<link rel="icon">` pointing to `favicon.svg`.
- `<link rel="stylesheet">` for `css/shimong.css`.
- RSS autodiscovery link (output format lookup).
- `<link href="https://fedi.ziyun.rocks/@ziyun" rel="me">` for fediverse verification.
- Calls `partial "seo.html"`.

### `layouts/partials/seo.html`

OpenGraph and Twitter Card tags:
- `og:title`, `og:site_name` (from `params.title`), `og:type` (`article` for pages,
  `website` otherwise), `og:url`.
- `twitter:card` = `summary`.
- `og:image` from `params.images` (first entry).
- `meta name="description"` from page `.Description` or `params.description` fallback.

### `layouts/partials/sidebar.html`

Single line: an `<a>` linking to the site root wrapping an `<h1>` containing
`.Site.Title`. CSS does the rest (see Styling).

### `layouts/partials/nav.html`

Iterates `$.Site.Menus.main`. Items are separated by ` &middot; ` (the `·` character).
Each item gets `class="active"` when `$currentPage.IsMenuCurrent "main" .` is true.
Schema.org `SiteNavigationElement` microdata attributes are present.

### `layouts/partials/footer.html`

One paragraph, italicised:
> Disclaimer: this blog may contain up to 100% LLM-generated content.

---

## Styling

**File:** `static/css/shimong.css`

Plain CSS — no SCSS, no Hugo Pipes. This is intentional: it avoids any dependency on
the Hugo extended build and makes the file editable without a compile step.

### Layout

`body` is `display:flex; flex-flow:row`. `body > header` (the sidebar) is naturally
sized. `#main` is `flex:auto; max-width:820px; display:flex; flex-flow:column`.

### Giant title trick

```css
body > header h1 {
  width: 1.5em;
  font-size: 10em;
  margin: 0 auto;
}
```

The two characters of `.Site.Title` ("子雲") are constrained to `1.5em` wide at `10em`
font size, forcing them to wrap into a vertical stack — the signature shimong look. The
surrounding `<a>` has `text-decoration:none`.

### Code blocks

```css
pre {
  font-family: 'Fira Code', 'Nanum Gothic Coding', monospace;
  background-color: #272822;
  color: #f8f8f2;
  padding: 15px;
  overflow: auto;
  tab-size: 2;
}
.highlight { overflow-x: auto; }
.highlight pre { margin: 0; }
```

Chroma (configured with `noClasses = true` and `style = "monokai"`) emits inline-styled
`<span>` elements inside a `<pre>`. The CSS ensures the block has the monokai dark
background and scrolls horizontally rather than overflowing the page. Inline code in
`p` and `li` gets a distinct magenta color (`#930d72`).

### Dark mode

```css
@media (prefers-color-scheme: dark) {
  html { background-color: black; color: #dfdfdf; }
  article img:not([src$=".jpg"]):not([src$=".jpeg"]):not(.opaque) {
    background-color: white;
    border: 5px solid white;
    box-sizing: border-box;
  }
  /* ... kbd adjustments ... */
}
```

PNG images (typically transparent) get a white background in dark mode. Add class
`opaque` to an image to opt out. JPEG images are excluded by file extension.

### Responsive collapse (≤640px)

`body` switches to `flex-direction:column`, stacking the sidebar above content. The
title shrinks from `font-size:10em` to `7em` and the `width:1.5em` constraint is
removed (`width:auto`), so it renders horizontally.

### Other notable rules

- `#main > main` has `margin-right:25px` (removed on mobile).
- `hr` is styled as a narrow 25px centred divider used between post body and comments.
- `.article-list article > header` is a flex row aligning the date and title inline.
- `blockquote` has a 4px left border in `#ccc`.
- `iframe.video` is full-width at 460px height (260px on mobile).

---

## Configuration (`config.toml`)

```toml
baseURL  = "https://blog.ziyun.rocks"
title    = "子雲"          # renders as the giant sidebar text
locale   = "en-US"
enableRobotsTXT = true
enableGitInfo   = true     # provides git-derived dates

[frontmatter]
date    = ["date", "publishDate", "lastmod"]
lastmod = ["lastmod", ":git", "date", "publishDate"]
# lastmod falls back to the git commit date if not set in front matter

[params]
description = "Ziyun = :metal:"
favicon     = "favicon.svg"
images      = ["images/share.png"]   # OG/Twitter share image
title       = "Ziyun Rocks"          # og:site_name (different from .Site.Title)
dateFormat  = "2006-01-02"

[taxonomies]
  tag = "tags"     # enables /tags/ and per-tag /tags/<tag>/ pages

[markup.highlight]
  style     = "monokai"
  noClasses = true      # inline styles; no separate CSS class file needed
  codeFences = true     # fenced code blocks in Markdown are highlighted

[[menu.main]]
  name = "blog"   url = "/"       weight = 1
[[menu.main]]
  name = "about"  url = "/about/" weight = 2
[[menu.main]]
  name = "tags"   url = "/tags/"  weight = 3
```

**`[markup.highlight]` note:** `noClasses = true` means Chroma writes syntax colours as
inline `style` attributes. Combined with the CSS `pre` background and `overflow-x:auto`,
fenced code blocks are self-contained and horizontally scrollable.

**`enableGitInfo` + `[frontmatter] lastmod`:** Hugo uses the last git commit date for a
file as its `lastmod` if the front matter does not explicitly set it. Netlify also sets
`HUGO_ENABLEGITINFO = "true"` in `netlify.toml`.

---

## Content & URLs

- Posts live in `content/posts/` as either single `.md` files or page-bundle
  directories (a directory named `YYYY-MM-DD-slug/` with an `index.md` and any
  co-located images).
- Post URLs follow the default Hugo permalink: `/posts/<slug>/`. The `slug` front
  matter field overrides the filename-derived slug (e.g., `slug: compile-redis-with-zig`).
- `content/posts/_index.md` has `title: "Archive"` (used as the section title in
  `list.html`).
- `content/about.md` renders at `/about/`.
- Posts with `draft: true` in front matter are excluded from production builds. Netlify
  deploy-preview contexts use `--buildFuture` but not `--buildDrafts`.

---

## Customizing

**Add or reorder a nav item** — edit `[[menu.main]]` entries in `config.toml`. The
`weight` field controls order (ascending). Add a new entry with `name`, `url`, and
`weight`.

**Change the syntax-highlight theme** — set `style` under `[markup.highlight]` in
`config.toml`. Valid values are Chroma style names (e.g., `dracula`, `github`, `monokai`).
Also update the `background-color` and `color` in the `pre` block in
`static/css/shimong.css` to match the new theme's background and default foreground.

**Change the sidebar title** — edit `title = "子雲"` at the top of `config.toml`. Any
string of roughly 1–3 characters will work well with the `width:1.5em` vertical-stack
trick; longer strings may need a CSS tweak.

**Adjust colors or dark mode** — edit the relevant rules in `static/css/shimong.css`
directly. The dark-mode block starts at `@media (prefers-color-scheme: dark)`. There is
no build step; changes take effect immediately on the next Hugo build.

**Add a new taxonomy** — add an entry to `[taxonomies]` in `config.toml` and add a
`[[menu.main]]` entry pointing to `/<taxonomy>/`. Hugo will generate the index and
per-term pages automatically using `_default/terms.html` and `_default/list.html`.
