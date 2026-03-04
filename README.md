# blog.anandvivek.in

Personal blog hosted at [blog.anandvivek.in](https://blog.anandvivek.in) via **Cloudflare Pages**.

Built with [Hugo](https://gohugo.io/) using the [Paper](https://github.com/nanxiaobei/hugo-paper) theme.

## Writing a new post

```bash
hugo new content posts/my-new-post/index.md
```

Edit the generated file, set `draft: false` when ready, then push.

## Local development

```bash
hugo server -D
# visit http://localhost:1313
```

## Cloudflare Pages build settings

| Field | Value |
|---|---|
| Build command | `hugo` |
| Build output directory | `public` |
| Environment variable | `HUGO_VERSION` = `0.145.0` |
