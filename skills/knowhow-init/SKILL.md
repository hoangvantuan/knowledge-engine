---
name: knowhow-init
description: "Khởi tạo hệ thống knowhow trong một kho tri thức. Tạo thư mục .knowhow/ với schema, wiki, skills registry, workflows registry. Dùng khi bắt đầu kho mới hoặc muốn thêm quản lý knowhow vào không gian làm việc hiện có. Trigger: 'knowhow init', 'khởi tạo knowhow', 'thêm knowhow', 'setup knowhow', hoặc khi user muốn bắt đầu ghi nhận tri thức của kho."
---

# Knowhow Init

## Tổng quan

Knowhow System là nghi thức tích luỹ tri thức có cấu trúc cho một kho (AI viết, người duyệt) qua 3 lớp:

| Lớp                    | Vai trò                                          |
| ---------------------- | ------------------------------------------------ |
| **Raw**                | Nguồn thô, immutable. Agent đọc nhưng không sửa. |
| **Wiki**               | Tri thức có cấu trúc, liên kết chéo.             |
| **Skills & Workflows** | Kiến thức thực thi, tái sử dụng.                 |


Phân biệt Skill và Workflow:

- **Skill** = thao tác khép kín, làm-theo-được, tái dùng cho task tương tự (được phép gắn domain).
- **Workflow** = chuỗi bước, gắn domain, gọi nhiều skill. Thay đổi khi đổi domain.

## Flow khởi tạo

### Bước 1: Thu thập thông tin

Hỏi user 2 câu:

1. **Tên kho tri thức** (dùng cho SCHEMA.md và log)
2. **Mô tả lĩnh vực** (1-2 câu, giải thích kho phục vụ việc gì)

Nếu user cung cấp sẵn trong prompt, không cần hỏi lại.

### Bước 2: Tạo cấu trúc thư mục

Tạo toàn bộ cây thư mục `.knowhow/` tại root workspace:

```
.knowhow/
├── SCHEMA.md
├── schema-signals.md
├── raw/
├── inbox/
├── archive/
├── wiki/
│   ├── index.md
│   └── log.md
├── skills/
│   └── registry.md
└── workflows/
    └── registry.md
```

Chạy lệnh tạo thư mục:

```bash
mkdir -p .knowhow/{raw,inbox,archive/inbox,wiki,skills,workflows}
```

### Bước 3: Sinh SCHEMA.md

Đọc file `references/schema-template.md` trong thư mục skill này.

Thay thế:

- `{{PROJECT_NAME}}` → tên kho user cung cấp
- `{{PROJECT_DESCRIPTION}}` → mô tả domain user cung cấp
- `{{DATE}}` → ngày hiện tại (YYYY-MM-DD), dùng cho entry Changelog đầu tiên trong SCHEMA

Ghi kết quả vào `.knowhow/SCHEMA.md`.

### Bước 4: Sinh các file nội dung

**wiki/index.md**:

```markdown
# Wiki Index

## Decision

## Pattern

## Concept

## Troubleshooting
```

**wiki/log.md** (thay `YYYY-MM-DD` bằng ngày hiện tại, `{{PROJECT_NAME}}` bằng tên kho):

```markdown
# Activity Log

## [YYYY-MM-DD] init | Khởi tạo .knowhow/ cho kho {{PROJECT_NAME}}
```

**skills/registry.md**:

```markdown
# Skill Registry

| Skill | Mô tả | Version | Tags | Cập nhật |
|-------|--------|---------|------|----------|
```

**workflows/registry.md**:

```markdown
# Workflow Registry

| Workflow | Mô tả | Skills dùng | Version | Cập nhật |
|----------|--------|-------------|---------|----------|
```

**schema-signals.md** (top-level, cạnh SCHEMA.md, là sổ tích luỹ tín hiệu tiến hoá, tạo rỗng chỉ có header):

```markdown
# Schema Signals

Sổ tích luỹ tín hiệu "khuôn không vừa". Append-only, parse được.
distill và query ghi vào "Đang chờ xử lý". lint `schema-review` đọc, áp ngưỡng, rồi cắt tín hiệu đã dùng sang "Đã xử lý".

Format mỗi dòng:
`- [YYYY-MM-DD] <nguồn> | <loại> | <chi tiết ngắn> | related: <slug-hoặc-tag>`

- nguồn ∈ distill | query | run
- loại ∈ no-fit-type | adhoc-section | query-miss | promote-candidate
- promote-candidate: tín hiệu "một wiki pattern/troubleshooting page đang bị dùng lặp để LÀM THEO", ứng viên nâng thành skill/workflow. query/run phát; `related` trỏ slug page nguồn. Tổng hợp promote-candidate là việc của `knowhow-distill` (KHÔNG phải schema-review); schema-review BỎ QUA các dòng promote-candidate.
- lint scan (type-bloat, tag-cluster) KHÔNG ghi vào sổ này, tính live khi schema-review chạy.

## Đang chờ xử lý

## Đã xử lý
```

### Bước 5: Thêm hướng dẫn vào agent config

Tìm file cấu hình agent tại root workspace theo thứ tự ưu tiên:

1. `CLAUDE.md`
2. `AGENTS.md`
3. `GEMINI.md`

Quy tắc:

- Thêm vào file **đầu tiên** tìm thấy.
- Nếu **chưa có file nào**, tạo `CLAUDE.md` mới.
- **Idempotent**: TRƯỚC khi append, grep `## Knowhow` trong file config. Nếu đã có → BỎ QUA, không append lần hai (tránh nhân đôi khi init chạy lại):
  ```bash
  grep -q "^## Knowhow" <config-file> && echo "Đã có, bỏ qua" || cat references/agent-config-snippet.md >> <config-file>
  ```
- Nội dung thêm: đọc `references/agent-config-snippet.md` và append vào cuối file.

### Bước 6: Ghi log hoàn tất

Xác nhận với user:

- Liệt kê các file đã tạo
- Nhắc user đọc `.knowhow/SCHEMA.md` để hiểu quy tắc vận hành
- Gợi ý bước tiếp theo: capture knowhow đầu tiên vào `inbox/`
