# CLOUDFLARE + GITHUB STACK — Quy tắc kỹ thuật

## Stack chuẩn

- **Cloudflare Pages** — host static files (HTML/CSS/JS), auto-deploy từ GitHub
- **Cloudflare Pages Functions** → `_worker.js` — xử lý API, serverless logic
- **Cloudflare KV** — key-value storage (config, cache, user data)
- **Cloudflare D1** — SQLite database (nếu cần relational data)
- **Cloudflare R2** — object storage (file upload, images)
- **GitHub** — version control, trigger deploy
- **GitHub Actions** — cron jobs, CI scripts (Python/JS)

---

## _worker.js — Constraints bắt buộc

### KHÔNG dùng:
- `require()`, `fs`, `path`, bất kỳ Node.js built-in module
- `console.log` nhiều (tốn CPU trong production)
- Vòng lặp `await` tuần tự cho nhiều KV/fetch calls song song

### PHẢI dùng:
- `fetch()` native Web API
- `env.KV_NAME.get/put/delete/list` cho KV
- `Promise.all()` khi cần nhiều async operations song song
- `ctx.waitUntil()` cho background tasks không block response

### Pattern KV đúng:
```javascript
// ❌ SAI — tuần tự, chậm 3x
for (const key of keys) { const v = await env.KV.get(key); }

// ✅ ĐÚNG — song song
const values = await Promise.all(keys.map(k => env.KV.get(k)));
```

### CORS — bắt buộc trong MỌI response:
```javascript
const CORS = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type,Authorization",
};
if (request.method === "OPTIONS") return new Response(null, { headers: CORS });
```

### Export pattern cho Pages Functions:
```javascript
export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    // route theo url.pathname
  }
};
```

---

## KV — Quy ước naming

Dùng format `PREFIX:id` hoặc `PREFIX_id`:
```
CONFIG:site          → cấu hình chung
USER:abc123          → data user
CACHE:page-home      → cache trang
SESSION:token-xyz    → session data
```

TTL cho data tạm: `{ expirationTtl: 3600 }` (giây)

---

## GitHub Actions — cấu trúc cơ bản

```yaml
name: Deploy / Cron Job
on:
  schedule:
    - cron: '0 */6 * * *'   # mỗi 6 tiếng
  workflow_dispatch:          # trigger thủ công
  push:
    branches: [main]
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pip install requests
      - run: python script.py
        env:
          API_KEY: ${{ secrets.API_KEY }}
```

## Secrets — 2 chỗ riêng biệt

| Dùng ở đâu | Set tại đâu |
|---|---|
| GitHub Actions | GitHub repo → Settings → Secrets → Actions |
| Cloudflare Worker runtime | Cloudflare Dashboard → Pages → Settings → Environment variables |

**Không đồng bộ tự động** — phải set thủ công cả 2 chỗ nếu cần cả 2.

---

## Deploy flow chuẩn

```
Push to GitHub main
  → Cloudflare Pages auto-detect → build → deploy (HTML + _worker.js)
  → GitHub Actions: chạy cron scripts nếu có
```

Cloudflare Pages không cần `wrangler.toml` cho Pages Functions đơn giản.
File `_worker.js` ở root → tự động thành Pages Function.
