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

## Bindings (KV, D1, R2, secrets, etc.)

**None are required** for this project: it is **static files only** (no `functions/` folder, no server code). There is nothing to bind a database or bucket to.

| Kind | Needed here? |
|------|----------------|
| **Environment variables / Secrets** | No — add later only if you introduce [Pages Functions](https://developers.cloudflare.com/pages/functions/) |
| **KV, D1, R2, Durable Objects, Queues** | No |
| **Service / Worker bindings** | No |

`wrangler.toml` in this repo sets **`pages_build_output_dir = "."`** and **does not declare bindings**, which matches a static site.

To sync settings from an existing Cloudflare project into a file (or pull dashboard bindings into git), use:

```bash
npx wrangler@latest pages download config fungible-site
```

Review Cloudflare’s docs before making the Wrangler file the [source of truth](https://developers.cloudflare.com/pages/functions/wrangler-configuration/#source-of-truth) for production.
