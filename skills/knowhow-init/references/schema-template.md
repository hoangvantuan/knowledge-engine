# Knowhow Schema: {{PROJECT_NAME}}

**schema_version**: 1

## Giới thiệu dự án
{{PROJECT_DESCRIPTION}}

## Cấu trúc thư mục

| Thư mục | Mục đích |
|---------|----------|
| raw/ | Nguồn thô, immutable. Agent đọc nhưng không sửa. |
| inbox/ | Bộ đệm chờ đúc kết. Nội dung chưa phân loại. |
| archive/ | Kho phục hồi (không xoá cứng): page rút khỏi wiki, cùng inbox item đã đúc kết hoặc bị bỏ qua. `archive/inbox/` giữ inbox item đã dọn. |
| wiki/ | Tri thức có cấu trúc, liên kết chéo. |
| skills/ | Thao tác cụ thể, làm-theo-được, tái sử dụng. |
| workflows/ | Chuỗi bước, gắn domain, gọi nhiều skill. |

> **Lưu ý file top-level**: ngoài `SCHEMA.md`, thư mục `.knowhow/` còn có `schema-signals.md`, là sổ tích luỹ tín hiệu "khuôn không vừa" (meta về khuôn, KHÔNG phải tri thức). distill và query ghi vào đây; lint `schema-review` đọc. File này KHÔNG nằm trong `wiki/` và KHÔNG vào `index.md`.

## Page Types

| Type | Đường dẫn file | Mục đích |
|------|----------------|----------|
| decision | wiki/decision-<slug>.md | Quyết định + lý do + bối cảnh |
| pattern | wiki/pattern-<slug>.md | Cách làm đã chứng minh hiệu quả |
| concept | wiki/concept-<slug>.md | Thuật ngữ, khái niệm riêng dự án |
| troubleshooting | wiki/troubleshooting-<slug>.md | Sự cố đã gặp + cách xử lý |
| skill | skills/<slug>.md | Thao tác cụ thể, làm-theo-được, tái sử dụng |
| workflow | workflows/<slug>.md | Chuỗi bước, gắn domain, gọi nhiều skill |

## Phân biệt Skill và Workflow

| | Skill | Workflow |
|---|---|---|
| Bản chất | Một thao tác khép kín, làm-theo-được | Chuỗi bước hoàn thành mục tiêu lớn hơn |
| Tái sử dụng | Dùng lại được cho task tương tự (có thể gắn domain) | Thấp hơn, gắn domain cụ thể |
| Phụ thuộc | Một thao tác khép kín (không nhất thiết độc lập hoàn toàn với kho) | Gọi nhiều skill, có thứ tự bước |
| Khi đổi domain | Ít thay đổi | Thay đổi nhiều |

## Naming Conventions

- File wiki: `wiki/<type>-<slug>.md`, slug `kebab-case`. Ví dụ: `wiki/decision-rest-to-graphql.md`, `wiki/pattern-retry-with-jitter.md`.
- Skill: `skills/<slug>.md`, slug động từ + danh từ (`skills/parse-invoice.md`, `skills/write-commit-msg.md`).
- Workflow: `workflows/<slug>.md`, slug danh từ mô tả quy trình (`workflows/release-checklist.md`).
- Khi một type được tách vào subfolder (do schema tiến hoá), subfolder đặt tên theo type và file VẪN giữ nguyên tên `<type>-<slug>.md`. Ví dụ: `wiki/experiment/experiment-ab-test.md`. Giữ prefix để resolve `[[slug]]` tìm đệ quy theo tên file vẫn đúng.

## Cross-referencing

- Dùng `[[slug]]` để liên kết giữa các page. `slug` = phần sau prefix type. Ví dụ file `wiki/decision-rest-to-graphql.md` có slug `rest-to-graphql`, link bằng `[[rest-to-graphql]]`.
- Nếu nhiều page khác type trùng slug, link phải kèm type để khử nhập nhằng: `[[decision-rest-to-graphql]]`.
- Workflow reference skill bằng `→ Dùng skill: [[skill-name]]`.
- Mọi page phải xuất hiện trong index.md hoặc registry.md tương ứng.
- **Type động**: tập type hợp lệ ĐỌC TỪ bảng "Page Types" ở trên, không cố định. Khi schema tiến hoá thêm type mới (ví dụ `experiment`), link `[[experiment-<slug>]]` resolve được ngay.
- **Subfolder**: khi một type bị tách vào subfolder (ví dụ `wiki/experiment/experiment-abc.md`), link vẫn viết `[[experiment-abc]]` (slug trần hoặc kèm type). Resolve tìm đệ quy trong `wiki/`, không chỉ ở mức phẳng.

## Vòng đời metadata

### status (mọi page, gồm skill/workflow)

- `active`: đang dùng. Mặc định khi tạo.
- `deprecated`: còn để tham khảo nhưng có cách mới tốt hơn. Distill set khi thay thế cách cũ.
- `archived`: lỗi thời, chuyển vào `archive/`. Lint consolidation set.

> **Khôi phục không cần git**: mọi thao tác phá huỷ chỉ move vào `archive/` + đổi `status`, không xoá cứng. Lấy lại: move file từ `archive/` về chỗ cũ, đổi `status` về `active`, rồi chạy `knowhow-lint rebuild-index`.

### confidence (chỉ 4 wiki type, skill/workflow dùng version)

Đếm theo **số entry trong phần Changelog** của page:
- 1 entry (mới tạo) → `low`.
- ≥2 entry → `medium`.
- ≥3 entry → `high`.

- capture set `low` cho item mới.
- distill nâng khi page được cập nhật lặp lại (grep trỏ về page cũ → CẬP NHẬT → thêm entry changelog).
- lint consolidation hạ 1 bậc khi `updated` cũ hơn 90 ngày và không có entry changelog mới trong khoảng đó.

## Quy tắc vận hành

- Mọi knowhow vào inbox trước, KHÔNG ghi thẳng vào wiki/skills/workflows
- Khi distill: đọc index.md + registry.md trước. Ưu tiên cập nhật cái cũ hơn tạo mới
- Mọi thay đổi ghi changelog cuối page
- Mọi hoạt động ghi log vào wiki/log.md
- Mỗi entry log dùng heading prefix parse được: `## [YYYY-MM-DD] <op> | <tiêu đề>` (op ∈ init/capture/distill/lint/query). Cho phép `grep "^## \[" wiki/log.md | tail -5` xem hoạt động gần nhất.

## Onboarding cho agent mới

1. Đọc file SCHEMA.md này
2. Đọc wiki/index.md để biết dự án có knowhow gì
3. Đọc skills/registry.md và workflows/registry.md để biết có gì dùng được
4. Khi gặp vấn đề, tra wiki/ trước khi tự suy luận
5. Khi cần thực hiện quy trình, tra skills/ và workflows/ trước

> **Phạm vi agent**: `.knowhow/` là markdown thuần, MỌI agent ĐỌC được. Nhưng 4 skill vận hành (init, capture, distill, lint) hiện viết cho Antigravity (Gemini). Agent khác (Claude Code, Codex) chỉ ĐỌC knowhow, chưa chạy được capture/distill/lint cho tới khi skill được port.

## Tiến hoá cấu trúc (schema evolution)

Khuôn này tự tiến hoá theo dự án. Cơ chế:

1. distill/query phát hiện "khuôn không vừa" → ghi tín hiệu vào `schema-signals.md` (KHÔNG tự đổi khuôn).
2. `knowhow-lint schema-review` đọc sổ + quét sống + áp ngưỡng → đề xuất diff lên SCHEMA.md.
3. User duyệt từng đề xuất.
4. Hệ migrate file bị ảnh hưởng, rewrite link, rebuild index, bump `schema_version`, ghi vào Changelog dưới đây.

Bốn loại thay đổi cấu trúc: thêm/đổi/nghỉ hưu page type, đổi layout (subfolder), đổi format page type, sửa mục SCHEMA. Mọi thay đổi reversible kép: gốc là `archive/` + `status` (không xoá cứng), `git revert` là lớp cộng khi kho có git.

## Glossary & Convention (tiến hoá)

[Thuật ngữ riêng dự án + quy ước vận hành bổ sung. Trống lúc init. schema-review thêm vào khi phát hiện thuật ngữ/quy ước lặp nhiều lần.]

## Changelog

- {{DATE}}: Khởi tạo schema v1 (init).
