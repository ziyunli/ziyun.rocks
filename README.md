# [Ziyun Rocks](https://blog.ziyun.rocks)

[![Netlify Status](https://api.netlify.com/api/v1/badges/ab9865d6-cd79-4ace-881a-3aac9d538a2d/deploy-status)](https://app.netlify.com/sites/cute-kataifi-dd101a/deploys)

Source for my personal blog — a [Hugo](https://gohugo.io) static site with a custom
in-repo theme ("shimong"), deployed to Netlify on every push to `main`.

## Quick start

Requires Hugo (extended). The version is pinned in [`netlify.toml`](netlify.toml)
(`HUGO_VERSION`) — match it locally. No `git submodule` step: the theme lives in the
repo (`layouts/` + `static/`), not a submodule.

```sh
# local preview, drafts included
hugo server -D

# serve over the local network
hugo server --bind 0.0.0.0 --baseURL http://<your-ip> --port 8080
```

## More

- **[AGENTS.md](AGENTS.md)** — working in this repo: commands, project layout, content
  authoring, conventions, and deployment.
- **[docs/theme.md](docs/theme.md)** — the shimong theme reference: templates, styles,
  and configuration.
