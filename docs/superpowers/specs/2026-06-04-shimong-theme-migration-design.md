# Shimong Theme Migration — Design

Date: 2026-06-04

## Goal

Replace the current Hugo "Bear" theme on the blog (https://blog.ziyun.rocks/) with
an adaptation of the **shimong** theme (originally a Zola theme,
https://github.com/ziyunli/zola-shimong, derived from emersion's shimong). The Bear
theme's code blocks render "unbound" (no background/scroll containment) and the
overall look is unsatisfying. Shimong gives a distinctive monospace look with a
giant vertical CJK title sidebar and properly contained dark code blocks.

## Decisions (settled during brainstorming)

- **Homepage = archive on home.** The giant title sits in the left sidebar on every
  page (including home), and the post list renders directly on the homepage. The
  shimong full-screen `#intro` splash is NOT used.
- **Nav = `blog · about · tags`.** This requires enabling a real `tags` taxonomy and
  scaffolding an about page.
- Build as **project-level `layouts/` + `static/`**, not a separate theme directory.
  Drop the `hugo-bearblog` theme entirely.

## Architecture

Shimong's layout is driven by its SCSS: `<body>` is `display:flex; flex-flow:row`.
Every page uses one consistent structure:

```
<body>                          (flex row)
  <header>                      left sidebar — giant vertical 子雲 (.Site.Title)
  <div id="main">               content column, max-width 820px
    <nav>                       blog · about · tags  (centered)
    <main>                      page content
    <footer>                    LLM disclaimer (+ comments live in <main> on posts)
  </div>
</body>
```

- The sidebar title uses `font-size:10em; width:1.5em`, so the two characters of
  `.Site.Title` ("子雲") wrap into a vertical stack — the signature shimong look.
- On screens ≤640px the body switches to `flex-direction:column`, collapsing the
  sidebar to the top with a smaller (7em) title.

## Components

### Templates (Tera → Hugo Go templates, in `layouts/`)

- `_default/baseof.html` — html shell + the `<body>` flex structure above; pulls in
  `head`, `sidebar`, `nav`, `footer` partials and the `main` block.
- `index.html` — home; renders the post archive list (reuses the article-list markup).
- `_default/single.html` — a post: title, date, tags, content, then the LLM
  disclaimer + utterances comments.
- `_default/list.html` — section/term listing (used for `/posts/` if hit directly and
  for `/tags/<tag>/`): the article-list of pages.
- `_default/terms.html` — the `/tags/` index: list of tags with counts.
- `404.html`.
- `partials/head.html` — meta, title, favicon, CSS link, RSS link, SEO/OpenGraph,
  `rel="me"` fedi link, X-Clacks-Overhead header.
- `partials/sidebar.html` — the giant-title link to home.
- `partials/nav.html` — renders `[[menu.main]]` items with `·` separators and an
  `active` class on the current page.
- `partials/footer.html` — the "may contain up to 100% LLM-generated content"
  disclaimer.
- `partials/seo.html` — OpenGraph/Twitter card tags (ported/trimmed from Bear).

### Styling

- `static/css/shimong.css` — shimong's `_theme.scss` hand-compiled to plain CSS,
  with these adaptations:
  - Drop the unused `#intro` splash rules.
  - Ensure code blocks are contained: `pre`, `.highlight`, and `.highlight pre` get
    the monokai dark background, padding, and `overflow-x:auto`.
  - Keep: monospace font stacks, `prefers-color-scheme` dark mode (black bg,
    `#dfdfdf` text), the transparent-PNG-on-dark white-background trick, `kbd`
    styling, blockquote, responsive collapse.
- Plain CSS served from `static/` (no Hugo Pipes / SCSS compilation), so it works
  regardless of whether Netlify's pinned Hugo 0.108.0 is the extended build.

### Code highlighting

- `config.toml` `[markup.highlight]`: `style = "monokai"`, `noClasses = true`,
  `codeFences = true`. Chroma then emits inline-styled `<pre>` blocks with a
  background; combined with the CSS `overflow-x:auto`/padding, fenced blocks are
  bounded and horizontally scrollable — fixing the current "unbound" problem.

### Config changes (`config.toml`)

- Remove `theme = 'hugo-bearblog'`.
- Remove `disableKinds = ["taxonomy"]` and the `ignoreErrors` taxonomy line.
- Add `[taxonomies]` with `tag = "tags"`.
- Add `[[menu.main]]` entries: blog (→ `/`), about (→ `/about/`), tags (→ `/tags/`).
- Leave post permalinks at the default so existing URLs (`/posts/<slug>/`) are
  unchanged.

### Content

- `content/about.md` — scaffold with placeholder body for the user to fill in.
- Existing posts untouched.

## Preserved behavior

Utterances comments on posts, the LLM-content disclaimer footer, the RSS feed,
`favicon.svg`, the `rel="me"` fediverse link, and the X-Clacks-Overhead
`GNU Terry Pratchett` header.

## Removed

- `themes/hugo-bearblog/` directory.
- The Bear-specific "Made with Hugo Bear" footer line.

## Testing / acceptance

Local `hugo` build is clean (no errors/warnings about missing layouts or taxonomy),
then a visual pass via `hugo server`:

1. Home renders the giant 子雲 sidebar + post archive list.
2. A post page renders title, date, content, and **code blocks have a dark
   background and scroll horizontally** (no overflow off the page).
3. Dark mode (via `prefers-color-scheme`) flips background to black with light text.
4. At ≤640px the sidebar collapses to the top and the title shrinks.
5. `/tags/` lists tags; `/tags/<tag>/` lists that tag's posts.
6. Existing post URLs (`/posts/<slug>/`) are unchanged.
7. Utterances comments load on a post.
8. RSS feed link is present and valid; 404 page renders.
9. `/about/` renders.

## Risks / notes

- Netlify pins `HUGO_VERSION = 0.108.0`. All template functions used (`where`,
  `range`, `.GetTerms`, `partial`, menus, taxonomies) exist in 0.108; plain-CSS in
  `static/` avoids any extended-build dependency. No version bump needed.
- Most existing posts have no tags; the tags index will be sparse until posts are
  tagged. This is acceptable per the nav decision.
