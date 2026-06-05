# Shimong Theme Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the Hugo "Bear" theme with an adaptation of the shimong theme — a monospace look with a giant vertical 子雲 title sidebar, archive-on-home, `blog · about · tags` nav, and properly contained dark code blocks.

**Architecture:** Build shimong as project-level `layouts/` + `static/` (no separate theme dir). Root layouts override the still-present Bear theme, so the site keeps building at every step; the Bear theme is removed only in the final config switch. CSS is shipped as plain CSS in `static/` (hand-compiled from shimong's SCSS) to avoid any dependency on Netlify's pinned Hugo 0.108.0 being the extended/SCSS build.

**Tech Stack:** Hugo (local extended 0.162; Netlify-pinned 0.108.0 — all functions used exist in 0.108), Go html/templates, Chroma syntax highlighting (monokai), plain CSS.

**Verification model:** No unit-test framework exists for a static-site theme. Each task is verified by `hugo` building clean and by `grep` against generated `public/` HTML. A final manual visual pass (Task 10) covers what only eyes can confirm. Build with drafts enabled during development so post pages exist to inspect: `hugo -D`.

---

### Task 1: Ship the shimong stylesheet

Hand-compiled from `zola-shimong/sass/_theme.scss`, with three adaptations vs. the original: the unused `#intro` splash rules are dropped, code blocks (`pre` / `.highlight`) are given containment (`overflow`, padding, dark bg), and the monospace font is applied site-wide.

**Files:**
- Create: `static/css/shimong.css`

- [ ] **Step 1: Write the stylesheet**

```css
:root {
  --font-family: "Menlo", "Monaco", "Lucida Console", "Liberation Mono", "DejaVu Sans Mono", "Bitstream Vera Sans Mono", "Courier New", monospace, serif;
}

/* Layout */
body {
  font-family: var(--font-family);
  display: flex;
  flex-flow: row;
  margin: 0;
}

body > header {
  text-align: center;
}

body > header h1 {
  width: 1.5em;
  font-size: 10em;
  margin: 0 auto;
}

body > header a {
  text-decoration: none;
}

#main {
  flex: auto;
  min-height: 100vh;
  max-width: 820px;
  overflow: hidden;
  display: flex;
  flex-flow: column;
}

#main > nav {
  padding: 15px;
  text-align: center;
}

#main > main {
  padding: 15px;
  flex: 1;
  margin-right: 25px;
}

#main > footer {
  padding: 15px;
  padding-top: 50px;
}

hr {
  border: none;
  border-top: 1px solid gray;
  max-width: 25px;
  margin: 35px auto;
}

.muted {
  color: gray;
}

/* Typography */
article > header {
  margin-bottom: 2em;
}

article > header time {
  color: gray;
  white-space: nowrap;
}

article img {
  max-width: 100%;
}

article video {
  max-width: 100%;
}

article .footnotes {
  font-size: 0.75em;
}

/* Code blocks — contained, scrollable, monokai */
pre {
  font-family: 'Fira Code', 'Nanum Gothic Coding', monospace;
  background-color: #272822;
  color: #f8f8f2;
  padding: 15px;
  overflow: auto;
  tab-size: 2;
  -moz-tab-size: 2;
}

.highlight {
  overflow-x: auto;
}

.highlight pre {
  margin: 0;
}

p code,
li code {
  color: #930d72;
}

a {
  color: inherit;
}

a:visited {
  color: inherit;
}

kbd {
  display: inline-block;
  padding: 3px 5px;
  font-size: 0.8em;
  line-height: 10px;
  vertical-align: middle;
  border: 1px solid black;
  border-radius: 3px;
  box-shadow: inset 0 -1px 0 black;
  background-color: rgba(0, 0, 0, 0.15);
}

iframe.video {
  border: none;
  width: 100%;
  height: 460px;
}

blockquote {
  border-left: 4px solid #ccc;
  margin: 1.5em 10px;
  padding: 0.5em 10px;
}

/* Article list */
.article-list article > header {
  display: flex;
  flex-flow: row;
  align-items: center;
  margin-bottom: 0;
}

.article-list h3 {
  display: inline-block;
  margin: 0.5em;
}

.article-list small {
  font-weight: normal;
  color: gray;
}

.alt-links {
  float: right;
  color: gray;
  margin: 30px 0;
}

/* Responsive */
@media (max-width: 640px) {
  body {
    flex-direction: column;
  }
  body > header h1 {
    width: auto;
    font-size: 7em;
  }
  #main {
    display: block;
    min-height: 0;
  }
  #main > main {
    margin-right: 0;
  }
  iframe.video {
    height: 260px;
  }
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  html {
    background-color: black;
    color: #dfdfdf;
  }
  article img:not([src$=".jpg"]):not([src$=".jpeg"]):not(.opaque) {
    background-color: white;
    border: 5px solid white;
    box-sizing: border-box;
  }
  kbd {
    border-color: white;
    background-color: rgba(255, 255, 255, 0.15);
    box-shadow: inset 0 -1px 0 white;
  }
}
```

- [ ] **Step 2: Verify the file exists and contains the key rules**

Run: `test -f static/css/shimong.css && grep -c "flex-flow: row" static/css/shimong.css && grep -c "#272822" static/css/shimong.css`
Expected: prints `1` then `1` (both rules present).

- [ ] **Step 3: Verify the site still builds (CSS is unreferenced so far — sanity check)**

Run: `hugo -D --quiet && echo BUILD_OK`
Expected: `BUILD_OK` with no error output.

- [ ] **Step 4: Commit**

```bash
git add static/css/shimong.css
git commit -m "feat: add shimong stylesheet"
```

---

### Task 2: Head and SEO partials

Replaces Bear's head/seo. Keeps the X-Clacks-Overhead Pratchett header, favicon, RSS link, and the `rel="me"` fediverse link. These partials are not referenced until Task 4's baseof, so the build is unaffected.

**Files:**
- Create: `layouts/partials/head.html`
- Create: `layouts/partials/seo.html`

- [ ] **Step 1: Write `layouts/partials/head.html`**

```go-html-template
<meta http-equiv="X-Clacks-Overhead" content="GNU Terry Pratchett" />
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
<title>{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{ . }} | {{ end }}{{ .Site.Title }}{{ end }}</title>
<link rel="icon" href="{{ "favicon.svg" | relURL }}" type="image/svg+xml">
<link rel="stylesheet" href="{{ "css/shimong.css" | relURL }}">
{{ with .OutputFormats.Get "rss" -}}
{{ printf `<link rel="%s" type="%s" href="%s" title="%s" />` .Rel .MediaType.Type .Permalink $.Site.Title | safeHTML }}
{{- end }}
<link href="https://fedi.ziyun.rocks/@ziyun" rel="me">
{{ partial "seo.html" . }}
```

- [ ] **Step 2: Write `layouts/partials/seo.html`**

```go-html-template
<meta name="description" content="{{ with .Description }}{{ . }}{{ else }}{{ .Site.Params.description }}{{ end }}">
<meta property="og:title" content="{{ if .IsHome }}{{ .Site.Title }}{{ else }}{{ .Title }}{{ end }}" />
<meta property="og:site_name" content="{{ with .Site.Params.title }}{{ . }}{{ else }}{{ .Site.Title }}{{ end }}" />
<meta property="og:type" content="{{ if .IsPage }}article{{ else }}website{{ end }}" />
<meta property="og:url" content="{{ .Permalink }}" />
<meta name="twitter:card" content="summary" />
{{ with .Site.Params.images }}{{ range first 1 . }}<meta property="og:image" content="{{ . | absURL }}" />{{ end }}{{ end }}
```

- [ ] **Step 3: Verify the build still succeeds (partials unreferenced so far)**

Run: `hugo -D --quiet && echo BUILD_OK`
Expected: `BUILD_OK`.

- [ ] **Step 4: Commit**

```bash
git add layouts/partials/head.html layouts/partials/seo.html
git commit -m "feat: add shimong head and seo partials"
```

---

### Task 3: Sidebar, nav, and footer partials

The sidebar is the giant-title `<header>`. Nav renders `[[menu.main]]` (added in Task 8) — until then `.Site.Menus.main` is empty and nav renders nothing, which is fine. Footer is the LLM-content disclaimer (the Bear "Made with" line is intentionally dropped). This **overwrites** the existing Bear `footer.html`.

**Files:**
- Create: `layouts/partials/sidebar.html`
- Create: `layouts/partials/nav.html`
- Modify (overwrite): `layouts/partials/footer.html`

- [ ] **Step 1: Write `layouts/partials/sidebar.html`**

```go-html-template
<a href="{{ .Site.Home.RelPermalink }}"><h1>{{ .Site.Title }}</h1></a>
```

- [ ] **Step 2: Write `layouts/partials/nav.html`**

```go-html-template
{{ $currentPage := . }}
<nav itemscope itemtype="http://schema.org/SiteNavigationElement">
{{- range $i, $e := .Site.Menus.main }}
{{- if $i }} &middot; {{ end }}
<a itemprop="url" class="{{ if $currentPage.IsMenuCurrent "main" . }}active{{ end }}" href="{{ .URL | relLangURL }}"><span itemprop="name">{{ .Name }}</span></a>
{{- end }}
</nav>
```

- [ ] **Step 3: Overwrite `layouts/partials/footer.html`**

```go-html-template
<p>
  <i>
    Disclaimer: this blog may contain up to 100% LLM-generated content.
  </i>
</p>
```

- [ ] **Step 4: Verify the build still succeeds**

Run: `hugo -D --quiet && echo BUILD_OK`
Expected: `BUILD_OK`.

- [ ] **Step 5: Commit**

```bash
git add layouts/partials/sidebar.html layouts/partials/nav.html layouts/partials/footer.html
git commit -m "feat: add shimong sidebar, nav, footer partials"
```

---

### Task 4: Base template

The flex `<body>` structure that defines the shimong look: sidebar `<header>` + `#main` column (nav, main block, footer). Overrides the Bear theme's `baseof.html`. After this, the still-present Bear `index.html`/`single.html` content blocks render inside the new shell (intermediate hybrid state — acceptable).

**Files:**
- Create: `layouts/_default/baseof.html`

- [ ] **Step 1: Write `layouts/_default/baseof.html`**

```go-html-template
<!DOCTYPE html>
<html lang="{{ with .Site.LanguageCode }}{{ . }}{{ else }}en-US{{ end }}">
<head>
  {{- partial "head.html" . -}}
</head>
<body>
  <header>
    {{- partial "sidebar.html" . -}}
  </header>
  <div id="main">
    {{- partial "nav.html" . -}}
    <main>
      {{- block "main" . }}{{ end -}}
    </main>
    <footer>
      {{- partial "footer.html" . -}}
    </footer>
  </div>
</body>
</html>
```

- [ ] **Step 2: Verify build and that the new shell is emitted**

Run: `hugo -D --quiet && grep -q 'id="main"' public/index.html && grep -q "shimong.css" public/index.html && echo SHELL_OK`
Expected: `SHELL_OK`.

- [ ] **Step 3: Commit**

```bash
git add layouts/_default/baseof.html
git commit -m "feat: add shimong base template"
```

---

### Task 5: Home archive template

The homepage lists every post in the `posts` section as a shimong article-list (date + title). Overrides the existing Bear `layouts/index.html`.

**Files:**
- Modify (overwrite): `layouts/index.html`

- [ ] **Step 1: Overwrite `layouts/index.html`**

```go-html-template
{{ define "main" }}
<div class="article-list">
  {{ range (where .Site.RegularPages "Section" "posts") }}
  <article>
    <header>
      <time datetime='{{ .Date.Format "2006-01-02" }}'>{{ .Date.Format (default "2006-01-02" .Site.Params.dateFormat) }}</time>
      <h3><a href="{{ .Permalink }}">{{ .Title }}</a></h3>
    </header>
  </article>
  {{ end }}
</div>
{{ end }}
```

- [ ] **Step 2: Verify the archive renders with post links**

Run: `hugo -D --quiet && grep -q 'class="article-list"' public/index.html && grep -q '/posts/' public/index.html && echo HOME_OK`
Expected: `HOME_OK`.

- [ ] **Step 3: Commit**

```bash
git add layouts/index.html
git commit -m "feat: home archive list in shimong style"
```

---

### Task 6: Single (post) template

Post page: title, date, tags, content, then an article footer with `<hr>` and utterances comments. The LLM disclaimer is NOT repeated here — it already renders site-wide via the baseof footer. Overrides the existing Bear `layouts/_default/single.html`.

**Files:**
- Modify (overwrite): `layouts/_default/single.html`

- [ ] **Step 1: Overwrite `layouts/_default/single.html`**

```go-html-template
{{ define "main" }}
<article>
  <header>
    <h1>{{ .Title }}</h1>
    {{ with .Date }}
    <time datetime='{{ .Format "2006-01-02" }}'>{{ .Format "2006-01-02" }}</time>
    {{ end }}
    {{ with .GetTerms "tags" }}
    &mdash;
    {{ range . }}<a href="{{ .Permalink }}">#{{ .LinkTitle }}</a> {{ end }}
    {{ end }}
  </header>

  {{ .Content }}

  <footer>
    <hr>
    <script src="https://utteranc.es/client.js"
            repo="ziyunli/ziyun.rocks"
            issue-term="pathname"
            label="comments"
            theme="preferred-color-scheme"
            crossorigin="anonymous"
            async>
    </script>
  </footer>
</article>
{{ end }}
```

- [ ] **Step 2: Verify a post page renders title and comments**

Run: `hugo -D --quiet && f=$(ls public/posts/*/index.html | head -1) && grep -q "<article>" "$f" && grep -q "utteranc.es" "$f" && echo POST_OK`
Expected: `POST_OK`.

- [ ] **Step 3: Commit**

```bash
git add layouts/_default/single.html
git commit -m "feat: post template in shimong style"
```

---

### Task 7: List and terms templates, and 404

`list.html` renders an article-list for any section/taxonomy-term page (used by `/tags/<tag>/`, and `/posts/` if hit directly). `terms.html` renders the `/tags/` index with counts. `404.html` is a minimal not-found page. None of these conflict with existing files; they fill in the layouts the Bear theme provided.

**Files:**
- Create: `layouts/_default/list.html`
- Create: `layouts/_default/terms.html`
- Create: `layouts/404.html`

- [ ] **Step 1: Write `layouts/_default/list.html`**

```go-html-template
{{ define "main" }}
<h1>{{ .Title }}</h1>
<div class="article-list">
  {{ range .Pages }}
  <article>
    <header>
      <time datetime='{{ .Date.Format "2006-01-02" }}'>{{ .Date.Format (default "2006-01-02" .Site.Params.dateFormat) }}</time>
      <h3><a href="{{ .Permalink }}">{{ .Title }}</a></h3>
    </header>
  </article>
  {{ end }}
</div>
{{ end }}
```

- [ ] **Step 2: Write `layouts/_default/terms.html`**

```go-html-template
{{ define "main" }}
<h1>{{ .Title }}</h1>
<ul>
  {{ range .Data.Terms.Alphabetical }}
  <li><a href="{{ .Page.Permalink }}">{{ .Page.Title }}</a> ({{ .Count }})</li>
  {{ end }}
</ul>
{{ end }}
```

- [ ] **Step 3: Write `layouts/404.html`**

```go-html-template
{{ define "main" }}
<h1>404</h1>
<p>This page wandered off. <a href="{{ .Site.Home.RelPermalink }}">Go home</a>.</p>
{{ end }}
```

- [ ] **Step 4: Verify build and 404 output**

Run: `hugo -D --quiet && grep -q "404" public/404.html && echo LIST_OK`
Expected: `LIST_OK`.

- [ ] **Step 5: Commit**

```bash
git add layouts/_default/list.html layouts/_default/terms.html layouts/404.html
git commit -m "feat: list, terms, and 404 templates"
```

---

### Task 8: Switch config to shimong and enable tags

Drops the Bear theme reference and the taxonomy-disabling hacks, enables a real `tags` taxonomy, adds the `blog · about · tags` menu, and configures monokai code highlighting. Post permalinks are left at the default (the `[permalinks]` block is removed because its `blog`/`tags` keys were unused and would otherwise route tag pages under `/blog/`); existing `/posts/<slug>/` URLs are unchanged. After this task the Bear theme's layouts are no longer on the lookup path, so the root layouts from Tasks 4–7 are the only ones serving pages.

**Files:**
- Modify: `config.toml`

- [ ] **Step 1: Remove the theme line**

Delete this line from `config.toml`:

```toml
theme = 'hugo-bearblog'
```

- [ ] **Step 2: Remove the taxonomy-disabling lines**

Delete these two lines from `config.toml` (under `[frontmatter]`'s comment block, near the top-level keys):

```toml
disableKinds = ["taxonomy"]
ignoreErrors = ["error-disable-taxonomy"]
```

- [ ] **Step 3: Remove the `[permalinks]` block**

Delete the entire block:

```toml
[permalinks]
blog = "/blog/:year/:slug/"
tags = "/blog/:slug"
```

- [ ] **Step 4: Append taxonomy, highlight, and menu config**

Add to `config.toml` (top-level tables — place after the `[params]` block so the params table isn't accidentally extended):

```toml
[taxonomies]
  tag = "tags"

[markup]
  [markup.highlight]
    style = "monokai"
    noClasses = true
    codeFences = true

[menu]
  [[menu.main]]
    name = "blog"
    url = "/"
    weight = 1
  [[menu.main]]
    name = "about"
    url = "/about/"
    weight = 2
  [[menu.main]]
    name = "tags"
    url = "/tags/"
    weight = 3
```

- [ ] **Step 5: Verify build is clean without the theme and that nav + tags now render**

Run: `hugo -D --quiet && echo BUILD_OK && grep -q "itemprop=\"name\">blog" public/index.html && test -f public/tags/index.html && echo NAV_TAGS_OK`
Expected: `BUILD_OK` then `NAV_TAGS_OK`. (No "found no layout" or taxonomy warnings.)

- [ ] **Step 6: Verify code blocks are now containerized with a background**

Run: `f=$(ls public/posts/*/index.html | head -1); grep -q 'class="highlight"' public/posts/*/index.html && echo HIGHLIGHT_OK || echo "NOTE: no fenced code in sampled posts — check one with a code block"`
Expected: `HIGHLIGHT_OK` (the repo has posts with fenced code, e.g. the redis/zig and youtube-summary posts).

- [ ] **Step 7: Commit**

```bash
git add config.toml
git commit -m "feat: switch config to shimong, enable tags taxonomy and menu"
```

---

### Task 9: About page and remove the Bear theme

Scaffolds the about page (renders via `_default/single.html` at `/about/`) and deletes the now-unused Bear theme directory and leftover Bear partials.

**Files:**
- Create: `content/about.md`
- Delete: `themes/hugo-bearblog/` (whole directory)
- Delete: `layouts/partials/header.html` (Bear sidebar — replaced by `sidebar.html`)
- Delete: `layouts/partials/extend_head.html` (its `rel="me"` link moved into `head.html`)

- [ ] **Step 1: Write `content/about.md`**

```markdown
---
title: "About"
---

About page — content coming soon.
```

- [ ] **Step 2: Delete the Bear theme and leftover Bear partials**

```bash
git rm -r themes/hugo-bearblog
git rm layouts/partials/header.html layouts/partials/extend_head.html
```

- [ ] **Step 3: Verify the build is clean and `/about/` renders**

Run: `hugo -D --quiet && echo BUILD_OK && test -f public/about/index.html && grep -q "About" public/about/index.html && echo ABOUT_OK`
Expected: `BUILD_OK` then `ABOUT_OK`.

- [ ] **Step 4: Commit**

```bash
git add content/about.md
git commit -m "feat: add about page, remove bear theme"
```

---

### Task 10: Full build and manual visual verification

Confirms the acceptance criteria from the spec that require human eyes / a browser. Drafts ON so the in-progress posts are viewable.

**Files:** none (verification only).

- [ ] **Step 1: Production-style build is clean**

Run: `hugo --gc --minify -b https://blog.ziyun.rocks && echo PROD_BUILD_OK`
Expected: `PROD_BUILD_OK`, no warnings about missing layouts/taxonomies.

- [ ] **Step 2: Confirm existing post URLs are unchanged**

Run: `ls public/posts/ | head`
Expected: directories matching the current live slugs (e.g. `reboot-macbook-ollama`, the dated `running-openclaw-in-vagrant`, etc.) — same paths as before the migration.

- [ ] **Step 3: Serve and eyeball in a browser**

Run: `hugo server -D`
Then open http://localhost:1313/ and confirm:
  - Home: giant vertical 子雲 sidebar on the left, nav `blog · about · tags`, post archive list.
  - A post: title, date, content; **fenced code blocks have a dark monokai background and scroll horizontally instead of overflowing the page**.
  - Toggle OS dark mode (or DevTools "prefers-color-scheme: dark"): background goes black, text light; transparent PNGs get a white backing box.
  - Narrow the window to ≤640px: sidebar collapses to the top, title shrinks to ~7em.
  - `/tags/` lists tags; click a tag (use a tagged post if any exist) → its post list. `/about/` renders. A bad URL → the 404 page.
  - Utterances comments widget loads at the bottom of a post.

- [ ] **Step 4: Final commit if any tweaks were needed during the visual pass**

```bash
git add -A
git commit -m "fix: shimong visual polish"
```

(Skip if the visual pass needed no changes.)

---

## Notes for the implementer

- Build with `-D` during development so draft posts (e.g. the surviving-the-LLM-era post) exist to inspect; the production build/deploy excludes them as before.
- `where .Site.RegularPages "Section" "posts"` defaults to date-descending order — newest first, matching the current site.
- If `IsMenuCurrent` highlighting on the home/blog item looks off, it is cosmetic only (the `.active` class) and does not block completion.
- Do NOT change `[permalinks]` to "preserve" anything — removing it is what preserves the `/posts/<slug>/` URLs, because the old `blog`/`tags` keys never matched the `posts` section.
