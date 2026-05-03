# BACKEND PATTERNS — _worker.js + GitHub Actions

## Worker routing pattern chuẩn

```javascript
// _worker.js
export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    const { pathname } = url;

    // CORS preflight
    if (request.method === "OPTIONS") {
      return new Response(null, { headers: CORS });
    }

    // Static file passthrough (Cloudflare Pages tự serve)
    // Worker chỉ xử lý /api/* routes

    if (pathname.startsWith("/api/")) {
      return handleAPI(request, env, ctx, url);
    }

    return new Response("Not found", { status: 404 });
  }
};

async function handleAPI(request, env, ctx, url) {
  const path = url.pathname.replace("/api", "");
  const method = request.method;

  try {
    // Route table rõ ràng
    if (path === "/items"  && method === "GET")    return getItems(env);
    if (path === "/items"  && method === "POST")   return createItem(request, env);
    if (path === "/items"  && method === "DELETE") return deleteItem(request, env);
    if (path.startsWith("/items/") && method === "GET") {
      const id = path.split("/")[2];
      return getItem(env, id);
    }

    return json({ error: "Not found" }, 404);
  } catch (e) {
    return json({ error: e.message }, 500);
  }
}
```

## Helper functions chuẩn

```javascript
const CORS = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type,Authorization",
};

const json = (data, status = 200) =>
  new Response(JSON.stringify(data), {
    status,
    headers: { ...CORS, "Content-Type": "application/json" },
  });

const okResp = (msg = "ok") => json({ ok: true, msg });
const errResp = (msg, status = 400) => json({ error: msg }, status);
```

## KV CRUD pattern

```javascript
// Đọc array từ KV (với fallback)
async function kvGetArray(env, key) {
  try {
    const raw = await env.KV.get(key);
    return raw ? JSON.parse(raw) : [];
  } catch { return []; }
}

// Ghi array
const kvSetArray = (env, key, arr) => env.KV.put(key, JSON.stringify(arr));

// Ví dụ CRUD items
async function getItems(env) {
  const items = await kvGetArray(env, "ALL_ITEMS");
  return json(items);
}

async function createItem(request, env) {
  const body = await request.json();
  const items = await kvGetArray(env, "ALL_ITEMS");
  const newItem = { id: Date.now().toString(), ...body, createdAt: new Date().toISOString() };
  items.unshift(newItem);
  await kvSetArray(env, "ALL_ITEMS", items);
  return json(newItem, 201);
}

async function deleteItem(request, env) {
  const { id } = await request.json();
  let items = await kvGetArray(env, "ALL_ITEMS");
  items = items.filter(x => x.id !== id);
  await kvSetArray(env, "ALL_ITEMS", items);
  return okResp("deleted");
}
```

## Auth đơn giản (password check trong Worker)

```javascript
// Dùng cho admin panel cá nhân
const ADMIN_PW = env.ADMIN_PASSWORD || "changeme";

function checkAuth(request) {
  const auth = request.headers.get("X-Admin-Key");
  return auth === ADMIN_PW;
}

// Hoặc query param cho GET requests
// url.searchParams.get("key") === ADMIN_PW
```

## GitHub Actions — Python script mẫu

```python
import requests, os, json

API_BASE = os.environ.get("WORKER_DOMAIN", "https://yoursite.pages.dev")

def fetch_and_push():
    # Fetch data từ nguồn nào đó
    r = requests.get("https://api.example.com/data", timeout=30)
    r.raise_for_status()
    data = r.json()

    # Push lên Worker
    res = requests.post(
        f"{API_BASE}/api/sync",
        json={"data": data},
        headers={"X-Admin-Key": os.environ["ADMIN_KEY"]},
        timeout=20,
    )
    print(f"Push → HTTP {res.status_code}")

if __name__ == "__main__":
    fetch_and_push()
```

## Response format nhất quán

```javascript
// Luôn trả JSON với shape nhất quán
// Success:  { ok: true, data: ... }
// Error:    { error: "message" }
// List:     { items: [...], total: N }
```

## Background task (không block response)

```javascript
async function handleWithBackground(request, env, ctx) {
  // Trả response ngay
  const response = json({ ok: true });

  // Chạy background task sau
  ctx.waitUntil(doHeavyWork(env));

  return response;
}
```
