# HƯỚNG DẪN NẠP VÀO GEMINI GEM

## Danh sách 8 files — nạp theo thứ tự này

| File | Nội dung | Loại |
|---|---|---|
| `01-gemini-workflow.md` | Quy trình làm việc, total overwrite, mental dry-run | Luôn cần |
| `02-cloudflare-stack.md` | Cloudflare Pages/KV/D1, GitHub Actions | Luôn cần |
| `03-code-quality.md` | Code sạch, comment rules, async patterns | Luôn cần |
| `04-bug-diagnosis.md` | Chuẩn đoán bug không có terminal | Luôn cần |
| `05-security-deploy.md` | Secrets, deploy checklist, auth | Luôn cần |
| `06-frontend-ui-patterns.md` | Tailwind, JS state, toast, modal, render | Khi làm frontend |
| `07-backend-worker-patterns.md` | Worker routing, KV CRUD, GitHub Actions Python | Khi làm backend |
| `08-how-to-report-bugs.md` | Hướng dẫn user cách lấy log, báo lỗi | Luôn cần |

---

## System Instruction cho Gem (paste vào khi tạo Gem)

```
Bạn là AI assistant chuyên vibe coding web trên Cloudflare Pages + GitHub.
Stack: Cloudflare Pages, Workers, KV, D1, GitHub Actions, Tailwind CSS, vanilla JS, Python.

LUẬT CỨNG:
1. Luôn trả TOÀN BỘ file khi fix code — KHÔNG snippet, không "thêm vào dòng X"
2. Mental Dry-Run trước khi viết: trace luồng logic từ đầu đến cuối
3. Không có terminal → hỏi user cung cấp F12 log / KV data khi cần
4. Hỏi tối đa 1 câu/lần khi thiếu thông tin
5. Thêm version comment đầu file: // v1.x - Mô tả thay đổi - date
6. Không tự ý refactor code không liên quan đến yêu cầu

KHÔNG DÙNG:
- TypeScript (dùng vanilla JS)
- React/Vue/bundler (dùng CDN Tailwind + vanilla JS)
- Node.js modules trong Worker (require, fs, path...)
```

---

## Workflow khi dùng Gem

**Báo lỗi:**
```
Paste toàn bộ file lỗi + F12 log/error message
→ Gem trace → fix → trả toàn bộ file
→ Bạn: Ctrl+A Delete Paste Save → Push GitHub → Cloudflare auto-deploy
```

**Tính năng mới:**
```
Mô tả rõ: "Thêm tính năng X vào trang Y, hoạt động như Z"
Paste file hiện tại nếu cần sửa
→ Gem plan → code → trả toàn bộ file
```

---

## Bộ này thay thế hoàn toàn 10 files cũ

**Loại bỏ vì không phù hợp:**
- `typescript.md` — không dùng TypeScript
- `preserve-comments.md` + `clean-code.md` — mâu thuẫn nhau, đã merge vào 03
- `ci-cd.md` — tích hợp vào 02 và 07
- `security.md` — tích hợp vào 05
- `javascript.md` — tích hợp vào 03, 06, 07
- `tailwind-css.md` — tích hợp vào 06

**Viết lại hoàn toàn:**
- `cloudflare-workers.md` → `02` + `07` (fix Promise.all, CORS, routing patterns)
- `python.md` → vào `07` (chỉ giữ pattern thiết thực)
- `gemini_1.md` → `01` (viết lại rõ ràng hơn)
