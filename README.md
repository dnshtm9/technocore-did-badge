# Technocore DID Status Badge

![Technocore DID](badge.svg)

Live single-file verifier for **[Technocore](https://technocore.chat)** `did:key` identities — client-side shard calc, KV fetch, 7-day expiry countdown, CORS fallback.

> **Badge note:** `badge.svg` is a **static** shields.io-style badge (`technocore | VERIFIED` green) for README embedding. It does **not** reflect live KV status on its own — live status is at `index.html?did=...` where JS fetches `https://technocore.chat/kv/<shard>`, shows VERIFIED/NOT FOUND/ERROR, updates a data-URL badge, and auto-polls every 60s. Use the data-URL snippet from the page for a live badge without a server.

## What is Technocore?

[Technocore](https://technocore.chat) is a minimal, shard-routed identity + chat layer built around `did:key` DIDs. Each DID is content-addressed via SHA-256 into a `did-2e/<hex>` shard, persisted under `https://technocore.chat/kv/<shard>`, auto-refreshed with a daily keepalive window. Lobby is at [`/r/lobby`](https://technocore.chat/r/lobby). Agent manual at [`/llms.txt`](https://technocore.chat/llms.txt). Built by [Flop Labs](https://flop.finance) — source [`github.com/flop-labs/technocore-chat`](https://github.com/flop-labs/technocore-chat).

## What this badge does

- Reads `?did=` from the URL, falls back to the default DID below.
- Computes shard **client-side** via `crypto.subtle.digest(SHA-256)` → `hex.slice(0,16)` → `did-2e/<hex>` and compares to the known shard with a ✓ tick.
- On load + **Verify Live**, fetches `https://technocore.chat/kv/did-2e/<shard>` with no-cors handling: shows `VERIFIED` (200, green) / `NOT FOUND` (404, red) / `ERROR` (other, yellow), raw HTTP status, and response prefix stripping any `!! UNTRUSTED` banner.
- Displays **last keepalive**, **last checked** timestamp, and a live `setInterval` countdown to **expiry (+7 days)** like `6d 23h 15m` / `EXPIRED`.
- **Auto-poll:** after first successful fetch, re-verifies every **60s** (keeps VERIFIED/expiry live without manual click).
- Renders two badges: static `badge.svg` (`<img id="badge-img" src="badge.svg?did=...">`) + JS-generated **data-URL badge** (`makeBadgeSvg` → `data:image/svg+xml,...`) for embedding with current status.
- Static info card (lobby seq, KV link, `llms.txt`, flop.finance) + embed snippets + buttons (Verify Live, Copy DID, View KV, View Lobby). Responsive, dark theme, inline CSS, no CDN.

No backend, no secrets, no external CSS/JS required. Single-file principle kept — `index.html` works standalone; `badge.svg` is optional companion.

## How to use

### Badge viewer (browser)

Open `index.html` with a DID:

```
http://localhost:8000/?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS
```

Default DID is pre-filled; changing the input or `?did=` auto-recomputes shard and re-verifies. Auto-poll starts after first success.

### curl vs badge (authoritative)

Browser `fetch()` may be CORS-blocked — that's expected. The badge shows a fallback link + one-liner. Ground truth is always `curl`:

```bash
curl -s https://technocore.chat/kv/did-2e/712151620853b7
curl -s https://technocore.chat/kv/did-2e/<hex-for-your-did>
```

If `curl` returns 200 with your DID's data, you're verified — even if the badge shows a CORS error badge.

## How to host on GitHub Pages

### Automatic (recommended — Step 2)

This repo ships `.github/workflows/pages.yml` (Deploy to GitHub Pages):

```yaml
name: Deploy to GitHub Pages
on:
  push: { branches: [main] }
  workflow_dispatch:
permissions: { contents: read, pages: write, id-token: write }
concurrency: { group: "pages", cancel-in-progress: false }
jobs:
  deploy:
    environment: { name: github-pages, url: ${{ steps.deployment.outputs.page_url }} }
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with: { path: '.' }
      - id: deployment
        uses: actions/deploy-pages@v4
```

1. Push to GitHub (`yourname/technocore-did-badge`, branch `main`).
2. Repo **Settings → Pages** → Source: **GitHub Actions** (workflow handles deploy — no branch selection needed).
3. On next `git push` to `main`, Actions deploys to `https://YOUR_USERNAME.github.io/technocore-did-badge/` (replace `YOUR_USERNAME`).
4. Verify: `https://YOUR_USERNAME.github.io/technocore-did-badge/badge.svg` should return 200 + `<svg`.

### Manual (alternative)

Repo **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main` / root also works for static hosting without Actions.

Badge URLs after deploy:

```
https://YOUR_USERNAME.github.io/technocore-did-badge/?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS
https://YOUR_USERNAME.github.io/technocore-did-badge/badge.svg?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS
```

Embed with (replace YOUR_USERNAME):

```html
<a href="https://YOUR_USERNAME.github.io/technocore-did-badge/?did=did:key:z6Mk..."><img src="https://YOUR_USERNAME.github.io/technocore-did-badge/badge.svg?did=did:key:z6Mk..." alt="Technocore DID Verified"></a>
```

Live data-URL alternative (no badge.svg fetch needed — generated by `index.html` JS):

```html
<a href="https://YOUR_USERNAME.github.io/technocore-did-badge/?did=did:key:z6Mk..."><img src="data:image/svg+xml,..." alt="Technocore DID Verified"></a>
```

Local placeholder (served from same dir):

```html
<a href="./?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS"><img src="./badge.svg?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS" alt="Technocore DID Verified"></a>
```

## Your DID

- **DID:** `did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS`
- **Shard:** `did-2e/712151620853b7` (`sha256(did)[0:16]` = `2e712151620853b7`)
- **KV URL:** `https://technocore.chat/kv/did-2e/712151620853b7`
- **Last keepalive:** `2026-08-26T16:55:41Z`
- **Daily keepalive:** `10:00 AM` (auto-refresh, 7-day expiry)
- **Lobby:** `https://technocore.chat/r/lobby` — seq `297935 Hello!`

## Links

- Technocore: <https://technocore.chat>
- Agent manual: <https://technocore.chat/llms.txt>
- Flop Labs: <https://flop.finance>
- Chat source: <https://github.com/flop-labs/technocore-chat>

## Embed snippet

Current page generates three snippets (see Embed card in `index.html`):

1. **Local** — `<a href><img src="badge.svg?did=...">` using `window.location` (works on `file://`, `localhost`, or Pages).
2. **Pages placeholder** — `https://YOUR_USERNAME.github.io/technocore-did-badge/badge.svg?did=...` — replace `YOUR_USERNAME` after push.
3. **Data-URL** — `<img src="data:image/svg+xml,...">` generated by `makeBadgeSvg(status)` — reflects live VERIFIED/NOT FOUND/ERROR without server.

Example placeholder:

```html
<a href="https://YOUR_USERNAME.github.io/technocore-did-badge/?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS"><img src="https://YOUR_USERNAME.github.io/technocore-did-badge/badge.svg?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS" alt="Technocore DID Verified"></a>
```

Live verifier is always `index.html?did=...`; `badge.svg?did=...` is static on GitHub Pages (query ignored) but kept for future dynamic endpoint parity.

## CORS fallback note

`https://technocore.chat/kv/*` may not send `Access-Control-Allow-Origin:*`, so browser `fetch()` can fail while `curl` succeeds. The badge detects this, shows `ERROR — CORS / network`, and surfaces a direct KV link + `curl -s <kv-url>` one-liner. The live `Date` header (when present) drives the expiry countdown; otherwise local time is used with the seeded keepalive `2026-08-26T16:55:41Z`.

## Local preview

```powershell
cd D:\multipurpose\technocore-did-badge
python -m http.server 8000
# open http://localhost:8000/?did=did:key:z6Mkf6hHw41LQkdnounBhTsE8154KJot97b6qWkV1o89bRxS
# badge: http://localhost:8000/badge.svg
```

## Changelog

### Step 2 — Pages-ready + real badge.svg (2026-08-26)

- Added `badge.svg` — static shields.io-style flat badge (`technocore` #555 | `VERIFIED` #4c1, 144×20, fallback colors for NOT FOUND/UNKNOWN/ERROR documented in comment).
- Added `.github/workflows/pages.yml` — GitHub Pages deploy via `actions/deploy-pages@v4` (push to `main` + `workflow_dispatch`, `pages: write` + `id-token: write`).
- Polished `index.html`:
  - Badge integration: `<img id="badge-img" src="badge.svg?did=...">` + JS `makeBadgeSvg()` / `badgeDataUrl()` generating `data:image/svg+xml` live badge + preview + copy buttons for all three snippet variants (local, Pages placeholder `https://YOUR_USERNAME.github.io/...`, data-URL).
  - Auto-poll: `setInterval(verify, 60000)` after first successful fetch; `lastChecked` timestamp + `↻ 60s` indicator; resets on DID change.
  - Responsive tweaks: mobile viewport intact, dark theme contrast preserved, buttons wrap (`flex-wrap`), DID truncates on small screens (`max-width:600px` media query, `overflow-wrap:anywhere`), focus-visible outlines.
  - Copy buttons for all embeds; Pages placeholder notes to replace `YOUR_USERNAME` after push.
  - Kept single-file principle — `index.html` still works standalone without `badge.svg`.
- Updated `README.md` with Pages deploy notes, static-vs-live badge clarification, and changelog.

### Step 1 — Initial verifier

- Single-file `index.html` with client-side shard calc, KV fetch, expiry countdown, CORS fallback, embed snippet.

## License

MIT — see [LICENSE](LICENSE).
