# Knowhow Schema — {{PROJECT_NAME}}

## Giới thiệu dự án
{{PROJECT_DESCRIPTION}}

## Cấu trúc thư mục

| Thư mục | Mục đích |
|---------|----------|
| raw/ | Nguồn thô, immutable. Agent đọc nhưng không sửa. |
| inbox/ | Bộ đệm chờ đúc kết. Nội dung chưa phân loại. |
| wiki/ | Tri thức có cấu trúc, liên kết chéo. |
| skills/ | Công việc cụ thể, chạy độc lập, tái sử dụng cao. |
| workflows/ | Chuỗi bước, gắn domain, gọi nhiều skill. |

## Page Types

| Type | Đường dẫn file | Mục đích |
|------|----------------|----------|
| decision | wiki/decision-<slug>.md | Quyết định + lý do + bối cảnh |
| pattern | wiki/pattern-<slug>.md | Cách làm đã chứng minh hiệu quả |
| concept | wiki/concept-<slug>.md | Thuật ngữ, khái niệm riêng dự án |
| troubleshooting | wiki/troubleshooting-<slug>.md | Sự cố đã gặp + cách xử lý |
| skill | skills/<slug>.md | Công việc cụ thể, chạy độc lập, tái sử dụng |
| workflow | workflows/<slug>.md | Chuỗi bước, gắn domain, gọi nhiều skill |

## Phân biệt Skill và Workflow

| | Skill | Workflow |
|---|---|---|
| Bản chất | Một công việc cụ thể, khép kín | Chuỗi bước hoàn thành mục tiêu lớn hơn |
| Tái sử dụng | Cao, dùng ở nhiều bối cảnh | Thấp hơn, gắn domain cụ thể |
| Phụ thuộc | Chạy độc lập | Gọi nhiều skill, có thứ tự bước |
| Khi đổi domain | Ít thay đổi | Thay đổi nhiều |

## Naming Conventions

- File wiki: `wiki/<type>-<slug>.md`, slug `kebab-case`. Ví dụ: `wiki/decision-rest-to-graphql.md`, `wiki/pattern-retry-with-jitter.md`.
- Skill: `skills/<slug>.md`, slug động từ + danh từ (`skills/parse-invoice.md`, `skills/write-commit-msg.md`).
- Workflow: `workflows/<slug>.md`, slug danh từ mô tả quy trình (`workflows/release-checklist.md`).

## Cross-referencing

- Dùng `[[slug]]` để liên kết giữa các page. `slug` = phần sau prefix type. Ví dụ file `wiki/decision-rest-to-graphql.md` có slug `rest-to-graphql`, link bằng `[[rest-to-graphql]]`.
- Nếu nhiều page khác type trùng slug, link phải kèm type để khử nhập nhằng: `[[decision-rest-to-graphql]]`.
- Workflow reference skill bằng `→ Dùng skill: [[skill-name]]`.
- Mọi page phải xuất hiện trong index.md hoặc registry.md tương ứng.

## Quy tắc vận hành

- Mọi knowhow vào inbox trước, KHÔNG ghi thẳng vào wiki/skills/workflows
- Khi distill: đọc index.md + registry.md trước. Ưu tiên cập nhật cái cũ hơn tạo mới
- Mọi thay đổi ghi changelog cuối page
- Mọi hoạt động ghi log vào wiki/log.md

## Onboarding cho agent mới

1. Đọc file SCHEMA.md này
2. Đọc wiki/index.md để biết dự án có knowhow gì
3. Đọc skills/registry.md và workflows/registry.md để biết có gì dùng được
4. Khi gặp vấn đề, tra wiki/ trước khi tự suy luận
5. Khi cần thực hiện quy trình, tra skills/ và workflows/ trước
