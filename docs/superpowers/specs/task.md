# Task: Knowhow System

## Trạng thái implement

| Skill | Trạng thái | File |
|-------|-----------|------|
| knowhow-init | Done (v1.1) | skills/knowhow-init/SKILL.md |
| knowhow-capture | Done (v1.1) | skills/knowhow-capture/SKILL.md |
| knowhow-distill | Done (v1.1) | skills/knowhow-distill/SKILL.md |
| knowhow-lint | Done (v1.1) | skills/knowhow-lint/SKILL.md |

## Test end-to-end (chạy lại sau fix v1.1)

Chạy trên 1 dự án thử (thư mục tạm), kiểm từng tiêu chí:

1. **init**: chạy `knowhow-init`. Kiểm:
   - `.knowhow/` có `raw/ inbox/ archive/ wiki/ skills/ workflows/`.
   - `wiki/` PHẲNG, KHÔNG có subfolder decisions/patterns/concepts/troubleshooting.
   - SCHEMA.md mô tả path `wiki/<type>-<slug>.md`, có mục archive/.
   - Chạy init lần 2 → config KHÔNG bị nhân đôi `## Knowhow` (I).

2. **capture**: capture 1 hội thoại (không file ngoài). Kiểm:
   - Có file `raw/YYYY-MM-DD-slug.md` (provenance luôn lưu raw — C3).
   - Inbox frontmatter `source_file: raw/...`, không trống.
   - Item có `tags` (phục vụ grep — C1).

3. **distill**: chạy `knowhow-distill`. Kiểm:
   - Có chạy `grep -ril` tìm trùng trước khi quyết định (C1).
   - Page tạo ra ở `wiki/<type>-<slug>.md` phẳng (A).
   - Frontmatter có `status: active`, wiki có `confidence` (C4).
   - Changelog ghi `(source: raw/...)`, không `inbox/...` (C3).

4. **capture lần 2 cùng chủ đề + distill**: Kiểm distill grep ra page cũ → CẬP NHẬT, không tạo trùng. Confidence nâng `low → medium` (≥2 entry changelog — C4).

5. **lint**:
   - `knowhow-lint` quick: tạo 1 link `[[slug]]` cố tình hỏng → lint báo. Tạo link đúng → KHÔNG báo hỏng giả (B).
   - `knowhow-lint rebuild-index`: xoá 1 dòng trong index.md → rebuild sinh lại đủ (F).
   - Link slug trùng 2 type → lint báo ambiguous (A5).

## Tiêu chí pass

Tất cả mục 1-5 đúng như mô tả. Đối chiếu "Tiêu chí done cho v1.1" trong spec review.
