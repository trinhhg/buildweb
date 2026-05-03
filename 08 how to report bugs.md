# CÁCH BÁO LỖI HIỆU QUẢ — Để Gemini chuẩn đoán đúng

## Gemini không thể chạy code → cần data thực tế từ bạn

---

## Template báo lỗi chuẩn

```
Tính năng: [tên tính năng / trang]
Triệu chứng: [mô tả chính xác điều gì xảy ra]
Đã thử: [những gì bạn đã thử để fix]
Log/Data: [paste data bên dưới]
```

---

## Lấy data từ đâu

### Lỗi JS / UI
**F12 → Console tab → copy dòng đỏ:**
```
Uncaught TypeError: Cannot read properties of undefined (reading 'map')
    at renderItems (index.html:142)
```

### Lỗi API / fetch
**F12 → Network tab → click request đỏ → copy:**
```
URL:      https://mysite.pages.dev/api/items
Method:   POST
Status:   500
Response: {"error": "KV binding not found"}
```

### Lỗi GitHub Actions
**Actions tab → click run đỏ → copy log section bị lỗi:**
```
Run python script.py
Traceback (most recent call last):
  File "script.py", line 45, in main
    r.raise_for_status()
requests.exceptions.HTTPError: 403 Client Error
```

### Lỗi Cloudflare Deploy
**Cloudflare Dashboard → Pages → project → Deployments → click → View build log**

### Data bất thường trong KV
**Workers & Pages → KV → namespace → paste giá trị key bị nghi vấn**

---

## Nguyên tắc khi báo lỗi

- **Copy paste** thay vì gõ lại (tên hàm, tên biến, error message dễ sai khi gõ lại)
- **Chính xác** version/trạng thái: "sau khi click Save" / "chỉ xảy ra khi field rỗng"
- **Không cần giải thích nhiều** — paste log + mô tả ngắn là đủ
