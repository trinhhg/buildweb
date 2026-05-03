# QUY TRÌNH LÀM VIỆC VỚI GEMINI — Vibe Coding

## Nguyên tắc cốt lõi

Gemini không có terminal. Không thể chạy lệnh, không thể đọc file thực tế.
Bù lại bằng **Mental Dry-Run** — chạy thử logic trong đầu trước khi viết code.

---

## Rule 1 — Luôn trả toàn bộ file (Total Overwrite)

- **KHÔNG snippet**, không "thêm đoạn này vào dòng X"
- Mọi sửa đổi → trả toàn bộ nội dung file
- Dòng comment đầu file: `// v1.2 - Fix tên bug - date`
- User thực hiện: Ctrl+A → Delete → Ctrl+V → Save → Deploy

## Rule 2 — Yêu cầu đúng thông tin, không đoán

Khi thiếu context → hỏi ngay, KHÔNG giả định:

| Loại lỗi | Cần user cung cấp |
|---|---|
| UI sai / trang trắng | F12 → Console → copy dòng đỏ |
| API fail | F12 → Network → click request đỏ → copy Status + Response |
| Data sai | Paste giá trị thực từ dashboard / KV / DB |
| Deploy không ăn | GitHub Actions log tab |

## Rule 3 — Mental Dry-Run trước khi viết code

Với mọi bug hoặc tính năng mới, thực hiện theo thứ tự:

```
1. ĐỌC: Đọc kỹ yêu cầu + toàn bộ code hiện tại
2. TRACE: Luồng user action → JS → fetch() → Worker/API → DB → response → render
3. TÌM: Điểm gãy hoặc điểm cần thêm
4. HYPOTHESIS: "Lỗi ở X vì Y. Side effect Z có thể xảy ra."
5. FIX: Viết code, tự check không gây regression
6. VERIFY: "Nếu user làm A thì B có đúng không?"
```

## Rule 4 — Không tự ý refactor

Khi user paste code → KHÔNG tự ý xóa, rút gọn, đổi tên, hay refactor những phần không liên quan đến yêu cầu. Chỉ sửa đúng chỗ được yêu cầu.

## Rule 5 — Cấu trúc file dự án

Không tách nhỏ file trừ khi user yêu cầu. Cấu trúc điển hình:
```
index.html        — Trang chính (public)
admin.html        — Trang quản trị (nếu có)
_worker.js        — Cloudflare Pages Worker (API logic)
scripts/          — JS phụ nếu quá lớn
style.css         — CSS nếu tách riêng
```

## Rule 6 — Hỏi tối đa 1 câu mỗi lần

Khi thiếu thông tin → hỏi 1 câu duy nhất, câu quan trọng nhất.
KHÔNG hỏi 3-4 câu cùng lúc.
