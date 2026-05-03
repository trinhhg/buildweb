# CODE QUALITY — Quy tắc viết code sạch

## Quy tắc Comment (đã giải quyết mâu thuẫn)

**Rule dứt khoát:**
- ✅ GIỮ: Comment giải thích **lý do** (business logic, edge case, workaround)
- ✅ GIỮ: Comment section divider `// ── Tên section ──`
- ✅ GIỮ: Comment tiếng Việt giải thích nghiệp vụ
- ❌ XÓA: Code bị comment-out không có giải thích tại sao
- ❌ XÓA: Comment chỉ mô tả lại đúng những gì code đang làm

```javascript
// ✅ Giữ — giải thích lý do
// Liangxin đôi khi trả YAML thẳng thay vì base64 (UA filtering)
const isYAMLRaw = rawB64.includes("proxies:");

// ❌ Xóa — chỉ mô tả lại code
// Kiểm tra nếu rawB64 chứa "proxies:"
const isYAMLRaw = rawB64.includes("proxies:");
```

## Naming

- Functions: `camelCase`, tên động từ rõ ràng: `fetchLinks()`, `buildYaml()`, `processB64()`
- Constants: `UPPER_SNAKE`: `RENAME_DJJC`, `LIANGXIN_DNS`
- KV keys: `PREFIX_TOKEN` pattern: `MAP_xxx`, `ORIG_URL_xxx`

## Error handling trong Worker

```javascript
// Pattern chuẩn
try {
  const data = await env.KV.get("key");
  return new Response(data || "[]", { headers: CORS });
} catch (e) {
  return new Response(e.message, { status: 500, headers: CORS });
}
```

## Async trong JavaScript

Luôn dùng `Promise.all()` khi có nhiều async operations không phụ thuộc nhau:

```javascript
// ✅
const [links, orders, devices] = await Promise.all([
  apG("/api/links", []),
  apG("/api/orders", []),
  apG("/api/devices", []),
]);

// ❌ Chậm hơn 3x
const links = await apG("/api/links", []);
const orders = await apG("/api/orders", []);
const devices = await apG("/api/devices", []);
```

## HTML/CSS với Tailwind

- Dùng Tailwind utility classes từ CDN (`cdn.tailwindcss.com`)
- Không dùng Tailwind config riêng (không có build step)
- Custom CSS ngắn đặt trong `<style>` tag
- Full screen: `html, body { height: 100%; }`, main `w-full` không max-width

## Python (update_sub.py)

- Type hints cho function params
- Docstring ngắn ở đầu function quan trọng
- `try/except` với traceback cho lỗi network
- `print()` rõ ràng để đọc GitHub Actions log
