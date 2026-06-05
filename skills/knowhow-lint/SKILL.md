---
name: knowhow-lint
description: "Rà soát sức khoẻ hệ thống knowhow. Có 2 chế độ: quick (link hỏng, orphan, thiếu frontmatter) và consolidation (deep audit: mâu thuẫn nội dung, trùng lặp, lỗi thời, thiếu phủ, nhất quán thuật ngữ). Trigger: 'knowhow lint', 'rà soát knowhow', 'kiểm tra knowhow', 'consolidation', 'audit knowhow', hoặc khi knowhow đã tích luỹ nhiều cần dọn dẹp."
---

# Knowhow Lint

Rà soát sức khoẻ hệ thống knowhow. Hai chế độ: **quick** (mặc định) và **consolidation** (deep audit).

## Precondition

1. Xác định thư mục `.knowhow/` trong workspace hiện tại.
2. Kiểm tra `.knowhow/` tồn tại và chứa ít nhất 1 trong: `wiki/`, `skills/`, `workflows/`.
3. Nếu không tồn tại → thông báo user chưa có hệ thống knowhow, dừng.

## Chọn chế độ

- User không nói gì thêm, hoặc nói "quick", "lint" → chạy **Quick Lint**.
- User nói "consolidation", "deep", "audit" → chạy **Consolidation**.

---

## Chế độ 1: Quick Lint (mặc định)

Chạy nhanh, kiểm tra 5 hạng mục. Không đọc nội dung page, chỉ đọc metadata và link.

### 1a. Registry sync

- Đọc `wiki/index.md`: mỗi entry trỏ đến file → kiểm tra file tồn tại.
- File `.md` trong `wiki/` nhưng không có trong `wiki/index.md` → báo.
- Tương tự cho `skills/registry.md` và `workflows/registry.md`.

### 1b. Link integrity

- Tìm tất cả `[[...]]` trong body các page (wiki, skills, workflows).
- Resolve link thành file path.
- File không tồn tại → báo link hỏng, ghi rõ page nguồn và target.

### 1c. Frontmatter check

Mọi page phải có: `type`, `title`, `created`.

Thêm theo loại:
- **Skill**: `version`, `input`, `output`
- **Workflow**: `version`, `trigger`

Thiếu field nào → báo cụ thể file và field thiếu.

### 1d. Inbox backlog

- Liệt kê file trong `inbox/`.
- File có `created` hoặc mtime cũ hơn 7 ngày → cảnh báo tồn đọng.

### 1e. Workflow-skill dependency

- Workflow reference `[[skill-name]]` → kiểm tra skill đó có trong `skills/registry.md`.
- Không có → báo.

### Output format (Quick Lint)

```markdown
## Quick Lint Report - YYYY-MM-DD

### Registry không đồng bộ (N items)
- [ ] Mô tả vấn đề

### Link hỏng (N items)
- [ ] Page X → [[Y]] nhưng Y không tồn tại

### Thiếu frontmatter (N items)
- [ ] File Z thiếu field: version

### Inbox tồn đọng (N items)
- [ ] inbox/abc.md (N ngày trước)

### Workflow dependency (N items)
- [ ] Workflow W reference [[S]] nhưng skill S không tồn tại

✅ Không có vấn đề nào (nếu clean)
```

Tạo báo cáo dạng artifact. Trình bày cho user duyệt.

### Sau khi user duyệt

Fix các vấn đề:
- Registry thiếu → thêm entry vào index/registry.
- Registry thừa (file không tồn tại) → xoá entry.
- Link hỏng → xoá hoặc sửa link, hỏi user nếu không chắc target đúng.
- Frontmatter thiếu → bổ sung field với giá trị mặc định hợp lý.
- Inbox tồn đọng → nhắc user xử lý, không tự xoá.
- Workflow dependency → báo user review, không tự xoá reference.

---

## Chế độ 2: Consolidation (deep audit)

Đọc **NỘI DUNG** tất cả page trong `wiki/`, `skills/`, `workflows/`.

### Bước thực hiện

1. Đọc `references/consolidation-checklist.md` để nắm 8 hạng mục rà soát.
2. Chạy Quick Lint trước (gộp kết quả).
3. Đọc toàn bộ nội dung các page.
4. Rà soát theo 8 hạng mục trong checklist.
5. Tạo báo cáo consolidation.

### Output format (Consolidation)

```markdown
## Consolidation Report - YYYY-MM-DD

### Mâu thuẫn nội dung (N items)
- [ ] Page A nói X, Page B nói Y → đề xuất thống nhất

### Đề xuất gộp (N items)
- [ ] Page A + Page B có nội dung chồng chéo → gộp?

### Lỗi thời (N items)
- [ ] Page X không cập nhật từ DD/MM → review hoặc archive?

### Thiếu phủ (N items)
- [ ] Lĩnh vực Y chưa có knowhow

### Nhất quán thuật ngữ (N items)
- [ ] Khái niệm Z được gọi là A ở page P1, gọi là B ở page P2

### Workflow dependency outdated (N items)
- [ ] Workflow W dùng skill S version 1.0, nhưng S đã là version 2.0
```

Tạo báo cáo dạng artifact. Trình bày cho user duyệt.

### Sau khi user duyệt

Thực thi từng thay đổi đã được duyệt:
- Gộp page → tạo page mới, xoá page cũ, cập nhật registry và link.
- Thống nhất thuật ngữ → find-replace trên tất cả page liên quan.
- Archive page lỗi thời → chuyển vào `archive/`, xoá khỏi registry.
- Bổ sung page thiếu phủ → tạo stub page, thêm vào registry.
- Cập nhật workflow dependency → sửa version reference.

---

## Log

Ghi vào `wiki/log.md`:

```
- [lint] Quick lint: N vấn đề phát hiện, M đã fix
- [lint] Consolidation: N vấn đề phát hiện, M đã fix
```
