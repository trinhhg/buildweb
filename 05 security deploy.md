# BẢO MẬT VÀ DEPLOY — Thực hành an toàn

## Secrets Management

### GitHub Actions
```yaml
# .github/workflows/update.yml
env:
  WORKER_DOMAIN: ${{ secrets.WORKER_DOMAIN }}
  # Set tại: GitHub repo → Settings → Secrets and variables → Actions
```

### Cloudflare Pages/Workers
- Không đồng bộ từ GitHub tự động
- Set tại: Cloudflare Dashboard → Pages → project → Settings → Environment variables
- Biến khác nhau cho Preview và Production — set cả 2

### TUYỆT ĐỐI KHÔNG:
- Hardcode API key, password trong code
- Commit file có chứa credentials lên GitHub public
- Log token hoặc password trong console/print

## Admin authentication trong HTML

Pattern hiện tại (password check ở client-side) phù hợp cho use case cá nhân:
```javascript
// OK cho cá nhân — không expose dữ liệu nhạy cảm qua URL
if (password === "572007") { loginOK(); }
```

Nếu cần bảo mật cao hơn: dùng Cloudflare Access hoặc Basic Auth tại Worker level.

## CORS Worker

Luôn handle OPTIONS preflight:
```javascript
const CORS = { "Access-Control-Allow-Origin": "*", ... };
if (request.method === "OPTIONS") return new Response(null, { headers: CORS });
```

## Deploy checklist

Trước khi push code lên GitHub (auto-trigger Cloudflare Pages deploy):
1. ✅ Không có hardcoded credentials
2. ✅ CORS headers đầy đủ trong worker
3. ✅ Environment variables đã set trên Cloudflare dashboard
4. ✅ GitHub Actions secrets đã set
5. ✅ Test trên web preview trước khi production

## GitHub Actions — cron update_sub.py

```yaml
name: Update VPN Sub
on:
  schedule:
    - cron: '*/60 * * * *'  # mỗi 60 phút
  workflow_dispatch:         # trigger thủ công
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with: { python-version: '3.11' }
      - run: pip install requests pyyaml
      - run: python update_sub.py
        env:
          WORKER_DOMAIN: ${{ secrets.WORKER_DOMAIN }}
```

## Cloudflare KV TTL

Dùng TTL cho data tạm thời:
```javascript
// Dead MAP token — tự xóa sau 7 ngày
await env.KV.put("MAP_" + token, "DEAD_NODE", { expirationTtl: 604800 });

// KHÔNG dùng TTL cho: ALL_LINKS, ALL_ORDERS, SUB_DATA (cần persist)
```
