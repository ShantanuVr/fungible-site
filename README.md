# Fungible Systems — static site

## If the build log says `Executing user deploy command: npx wrangler deploy`

**That command is configured in the Cloudflare dashboard, not in git.** This repo does not set it.

**Preferred fix:** remove the deploy command (steps below).  
**Repo workaround:** `wrangler.toml` now includes `[assets] directory = "."` so `npx wrangler deploy` can complete (Workers static assets). You should still clear the dashboard setting so you use **Pages** normally, not a Worker deploy inside a Pages build.

Do this now:

1. [Cloudflare Dashboard](https://dash.cloudflare.com) → **Workers & Pages** → select **this** project.
2. **Settings** → **Builds & deployments** (names vary slightly; look for **Build** / **Build configuration**).
3. Locate **Deploy command** (or **Custom deploy command** / **Non-production deploy command**).
4. **Delete everything** in that field so it is **blank**. Save.
5. Trigger a new deployment.

Git-connected **Pages** uploads the build output automatically. You should **not** use `npx wrangler deploy` (that is for **Workers**, not Pages, and causes the “Missing entry-point” / asset errors you are seeing).

See also: `CLOUDFLARE_CLEAR_DEPLOY_COMMAND.txt` in this repo.

---

## GoDaddy page / “Website Builder” instead of this site

If `fungible.co.in` shows a **GoDaddy** banner or **DPS** (GoDaddy Website Builder), your **nameservers can still be Cloudflare** while **DNS records** send visitors to **GoDaddy’s servers**. A quick check: response headers may show `Server: DPS/...` and CSP mentioning `godaddy.com`.

**Fix:** In **Cloudflare** → **DNS** for `fungible.co.in`, **remove or replace** any record for **`@` (apex)** or **`www`** that points to **GoDaddy** (parking IPs, `sites.godaddy.com`, `parked-*.godaddy.com`, old A records from the registrar wizard, etc.).

Then point the site at **your** hosting only, for example:

- **Pages:** `www` → **CNAME** → `<project>.pages.dev` (same account as the zone — see Error 1014), and set up the apex per **Custom domains** in the Pages project (or a supported apex pattern).
- **Worker:** keep **Workers** → **Routes** like `*.fungible.co.in/*` → `fungible-site` **and** ensure DNS does **not** override that with records to GoDaddy.

Also turn off **GoDaddy forwarding / Website Builder publish** for this domain in GoDaddy if you still use their dashboard (optional once DNS is only in Cloudflare).

---

## Error 522 — “Connection timed out” (e.g. on `www.fungible.co.in`)

Cloudflare reached its edge OK, but the **origin path for that hostname failed** (diagram: **Host** = red). This is almost always **DNS or routing**, not your HTML repo.

Check in order:

1. **Pages deployment is live**  
   **Workers & Pages** → your project → **Deployments** → latest **Production** build must be **Success** (not failed).

2. **Custom domain is active**  
   Same project → **Custom domains** → `www.fungible.co.in` should be **Active** with a valid certificate (not stuck *Pending*).

3. **DNS for `www` — point at Pages, not at the apex** (common cause of 522)  
   **Websites** → **fungible.co.in** → **DNS** → the **`www`** record:

   - **Do not** use **`www` → CNAME → `fungible.co.in`** (apex). That indirect chain is easy to get wrong: the apex may be bound to a Worker/Pages route while **`www` is a different hostname** and may not resolve to the same service reliably.
   - **Do** set **`www` → CNAME → `<your-project>.pages.dev`** (exact hostname is shown under **Workers & Pages** → your project → **Custom domains** when `www.fungible.co.in` is added — e.g. `fungible-site.pages.dev`). Keep **Proxied** (orange cloud).
   - **Remove** any **A** / **AAAA** on `www` pointing at an old host IP (that also causes **522**).

4. **Worker route not breaking `www`**  
   If a **Worker** has a route on `www.fungible.co.in/*` and it calls an origin that times out, you can see 522. Temporarily **disable** that Worker or the route to test.

5. **SSL/TLS**  
   **SSL/TLS** → **Overview** → use **Full** or **Full (strict)** (normal for Pages + proxied DNS).

After DNS changes, allow a few minutes and retry (or check from another network).

---

## Error 1014 — “CNAME Cross-User Banned”

Cloudflare does **not** allow an **orange-cloud (proxied) CNAME** to `*.pages.dev` (or another customer’s Workers hostname) when that target lives in a **different Cloudflare account** than the zone where you edit DNS. You see this if **`fungible.co.in`** is managed in **Account A** but **`fungible-site`** (Pages) was created in **Account B** (e.g. another login / team).

**Fix (pick one):**

1. **Use one account for both**  
   Log into the Cloudflare account where **`fungible-site`** exists. Ensure **the same account** also has the **`fungible.co.in` zone** (or [move/add the domain](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-domain/) to that account). Then add **`fungible.co.in` / `www`** under **Workers & Pages** → **fungible-site** → **Custom domains** and use the DNS values / setup wizard Cloudflare shows.

2. **Or recreate Pages in the account that already owns the domain**  
   If the domain must stay in Account A, create the Pages project (and Git connection) **in Account A**, deploy, then attach custom domains there. Remove or ignore the old project in the other account.

3. **Don’t mix two “homes” for the same site**  
   If you attached **`fungible.co.in`** to a **Worker** (`fungible-site`) *and* also use **Pages** with `www` → `fungible-site.pages.dev`, simplify: for this repo, use **Pages only** — remove redundant Worker custom-domain bindings on the apex if Pages is the source of truth.

Official reference: [Error 1014](https://developers.cloudflare.com/support/troubleshooting/http-status-codes/cloudflare-1xxx-errors/error-1014/).

---

## Cloudflare Pages (Git integration)

Use **Cloudflare Pages** default Git deploy — **not** Workers `wrangler deploy`.

In your Pages project → **Settings** → **Builds & deployments**:

| Setting | Value |
|--------|--------|
| **Framework preset** | None (or static HTML) |
| **Build command** | *(leave empty)* |
| **Build output directory** | `/` (root) |
| **Deploy command** | *(must be empty — see section above)* |

`npx wrangler deploy` targets **Workers**, not Pages, and will fail on this static site.

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
