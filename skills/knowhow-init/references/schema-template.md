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

| Type | Thư mục | Mục đích |
|------|---------|----------|
| decision | wiki/decisions/ | Quyết định + lý do + bối cảnh |
| pattern | wiki/patterns/ | Cách làm đã chứng minh hiệu quả |
| concept | wiki/concepts/ | Thuật ngữ, khái niệm riêng dự án |
| troubleshooting | wiki/troubleshooting/ | Sự cố đã gặp + cách xử lý |
| skill | skills/ | Công việc cụ thể, chạy độc lập, tái sử dụng |
| workflow | workflows/ | Chuỗi bước, gắn domain, gọi nhiều skill |

## Phân biệt Skill và Workflow

| | Skill | Workflow |
|---|---|---|
| Bản chất | Một công việc cụ thể, khép kín | Chuỗi bước hoàn thành mục tiêu lớn hơn |
| Tái sử dụng | Cao, dùng ở nhiều bối cảnh | Thấp hơn, gắn domain cụ thể |
| Phụ thuộc | Chạy độc lập | Gọi nhiều skill, có thứ tự bước |
| Khi đổi domain | Ít thay đổi | Thay đổi nhiều |

## Naming Conventions

- File: `kebab-case.md`
- Skill: động từ + danh từ (`parse-invoice.md`, `write-commit-msg.md`)
- Workflow: danh từ mô tả quy trình (`release-checklist.md`, `customer-onboard.md`)
- Decision: mô tả quyết định (`rest-to-graphql.md`)
- Pattern: mô tả pattern (`retry-with-jitter.md`)

## Cross-referencing

- Dùng `[[page-slug]]` để liên kết giữa các page wiki
- Workflow reference skill bằng `→ Dùng skill: [[skill-name]]`
- Mọi page phải xuất hiện trong index.md hoặc registry.md tương ứng

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
