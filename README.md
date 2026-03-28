# Fungible Systems — static site

## Cloudflare Pages (Git integration)

Use **Cloudflare Pages**, not Workers `wrangler deploy`.

In your Pages project → **Settings** → **Builds & deployments**:

| Setting | Value |
|--------|--------|
| **Framework preset** | None (or static HTML) |
| **Build command** | *(leave empty)* |
| **Build output directory** | `/` (root) |
| **Deploy command** / **Non-production branch deploy command** | *(remove / leave empty)* — do **not** run `npx wrangler deploy` |

`npx wrangler deploy` targets **Workers** and bundles the whole repo (including `node_modules`), which hits the **25 MiB** asset limit and fails.

This repository is intentionally **plain static files** (`index.html`, `_headers`, …) with **no `package.json`**, so Cloudflare will not run `npm install` during the build.

## Optional: deploy from your laptop (CLI)

```bash
npx wrangler@latest pages deploy . --project-name=fungible-site
```

Use `wrangler pages deploy`, not `wrangler deploy`.
