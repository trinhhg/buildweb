# CHUẨN ĐOÁN VÀ SỬA BUG — Không terminal, không run code

## Nguyên tắc: Diagnose từ triệu chứng

Vì không chạy được lệnh, phải suy luận từ mô tả. Dùng bảng này:

| Triệu chứng | Root cause thường gặp | Cần hỏi thêm |
|---|---|---|
| Trang trắng / lỗi JS | Exception trong render, parse JSON fail | F12 Console → dòng đỏ cụ thể |
| API trả lỗi | CORS thiếu, route sai, auth fail, env var chưa set | Status code + Response body |
| Data không cập nhật | Cache cũ (localStorage / KV), fetch không chạy | Có thấy request trong Network tab không? |
| Deploy xong không thấy thay đổi | Build cache, sai branch, worker cũ | GitHub Actions log + Cloudflare deployment log |
| Fetch thất bại (403/401) | Token hết hạn, sai credentials, IP block | Copy chính xác URL + headers đang gửi |
| State mất sau F5 | State lưu trong JS variable, không persist | Đang lưu localStorage hay fetch từ API? |
| CORS error | Worker thiếu CORS headers, OPTIONS chưa handle | Copy chính xác error message trong Console |

---

## Trace bug theo luồng chuẩn

```
User action (click/submit)
  → JS handler
  → fetch() call
  → Worker nhận request
  → Worker đọc/ghi KV/DB
  → Worker trả response
  → JS nhận response
  → render() update DOM
```

**Hỏi user:** Luồng bị gãy ở bước nào? Có thể biết qua F12 Network.

---

## Pattern diagnose từng loại lỗi

### Lỗi UI không render
```
Nghi vấn: data = null/undefined → .map() crash
Hỏi: F12 Console có dòng đỏ không? Copy paste cho mình xem
```

### Lỗi API 500
```
Nghi vấn: Worker throw exception, chưa catch
Hỏi: Cloudflare Workers → Logs có error không?
     Hoặc: Response body có message gì?
```

### Lỗi data sai sau deploy
```
Nghi vấn: KV/DB có data cũ, code mới chưa migrate
Hỏi: Xem giá trị trong KV/DB thực tế là gì?
```

### Lỗi chỉ xảy ra trên production, localhost OK
```
Nghi vấn: Environment variable chưa set trên Cloudflare
Hỏi: Cloudflare Pages → Settings → Environment variables đã có chưa?
```

---

## Khi nhận bug report

Trước khi viết code fix, phải xác định:
1. **Reproduce được chưa?** Điều kiện nào trigger lỗi?
2. **Expected vs Actual?** Mong đợi gì, thực tế nhận gì?
3. **Scope?** Lỗi mọi lúc hay chỉ trong điều kiện cụ thể?

Nếu chưa đủ thông tin → hỏi 1 câu duy nhất, câu quan trọng nhất.
