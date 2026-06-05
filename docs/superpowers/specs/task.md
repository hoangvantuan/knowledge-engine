# Task: Knowhow System

## Trạng thái implement

| Skill | Trạng thái | File |
|-------|-----------|------|
| knowhow-init | Done (v1.1) | skills/knowhow-init/SKILL.md |
| knowhow-capture | Done (v1.1) | skills/knowhow-capture/SKILL.md |
| knowhow-distill | Done (v1.1) | skills/knowhow-distill/SKILL.md |
| knowhow-lint | Done (v1.1) | skills/knowhow-lint/SKILL.md |
| knowhow-query | Done (v1.2) | skills/knowhow-query/SKILL.md |

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

---

## Test end-to-end v1.2 (schema evolution)

Chạy trên 1 dự án nghiên cứu giả lập (thư mục tạm). Mỗi kịch bản kiểm một loại thay đổi.

1. **Living contract + signals (nền)**: chạy `knowhow-init`. Kiểm:
   - `.knowhow/SCHEMA.md` có `**schema_version**: 1`, mục `## Changelog` với entry đầu `- <ngày>: Khởi tạo schema v1 (init)`.
   - `.knowhow/schema-signals.md` tồn tại, có `## Đang chờ xử lý` + `## Đã xử lý`, CHƯA có dòng tín hiệu nào (file có header/mô tả nhưng 2 section rỗng tín hiệu).
   - SCHEMA có mục `## Glossary & Convention` và `## Tiến hoá cấu trúc`.

2. **Thêm type (luồng chính)**: capture nhiều item kiểu "experiment".
   - distill emit `no-fit-type` đúng format vào `schema-signals.md` mục "Đang chờ xử lý".
   - **Dưới ngưỡng**: với < 5 tín hiệu, chạy `knowhow-lint schema-review` → KHÔNG đề xuất thêm type (hiển thị `✅ Không đủ ngưỡng nào`).
   - **Vượt ngưỡng**: sau ≥ 5 tín hiệu cùng chủ đề, chạy `schema-review` → đề xuất type `experiment` xuất hiện.
   - Duyệt → reclassify TỪNG FILE MỘT (user duyệt mỗi file, KHÔNG đổi hàng loạt).
   - Kiểm sau migrate: SCHEMA bảng Page Types có dòng `experiment`; `schema_version` bump lên 2; Changelog SCHEMA có entry mới; naming `wiki/experiment-<slug>.md` đúng; `wiki/index.md` rebuild có heading `## Experiment` liệt kê page đã reclassify; resolve `[[slug]]` vẫn chạy, không link hỏng.

3. **Query làm tín hiệu**: lặp một câu query không trúng page sạch (≥ 3 page chắp vá) → `knowhow-query` emit `query-miss` vào sổ → `schema-review` tính vào ngưỡng thêm type/page.

4. **Đổi layout**: tạo > 30 page cùng một type → `schema-review` đề xuất subfolder → duyệt → move file vào `wiki/<type>/`, rewrite mọi `[[link]]` trỏ tới. Kiểm: link trong FILE KHÁC (không phải file bị move) vẫn resolve đúng sau split, không link hỏng.

5. **File ngược qua inbox**: `knowhow-query` ra câu trả lời tốt, user OK file → kiểm: item vào `inbox/` (KHÔNG vào thẳng `wiki/`); inbox item có `captured_from: query` và `source_file: raw/YYYY-MM-DD-query-<slug>.md`; raw lưu Q&A tại `raw/YYYY-MM-DD-query-<slug>.md`.

6. **Reversibility**: sau một migrate, `git revert` → kiểm hệ về trạng thái cũ sạch.

### Tiêu chí pass v1.2

- distill/query ghi tín hiệu đúng định dạng vào `schema-signals.md`, KHÔNG tự đổi khuôn.
- `schema-review` chỉ đề xuất khi vượt ngưỡng, không báo nhiễu dưới ngưỡng.
- Migrate giữ link không hỏng, index rebuild đúng, SCHEMA version + changelog cập nhật.
- Bất biến "mọi knowhow qua inbox trước" không bị phá bởi query.
- Mọi migrate reversible bằng git.
- Log dùng prefix parse được: `grep "^## \[" .knowhow/wiki/log.md` ra danh sách op.
