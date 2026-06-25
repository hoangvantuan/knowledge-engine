---
name: knowhow-init
description: "Khởi tạo hệ thống knowhow từ số không cho một kho. CHỈ dùng khi workspace CHƯA có thư mục .knowhow/ và user muốn tạo lần đầu. Kích hoạt: 'init knowhow', 'setup knowhow', 'khởi tạo knowhow', 'thêm knowhow vào dự án'; muốn lập một hệ thống có cấu trúc để ghi lại tri thức/bài học của team ở nơi hiện chưa có hạ tầng nào; team cứ mất cách giải quyết vì không có chỗ ghi. Tạo .knowhow/ gồm wiki, registry skill, registry workflow, và gắn hướng dẫn vào config agent. KHÔNG dùng khi .knowhow/ đã tồn tại, hay khi khởi tạo hạ tầng dự án khác (git, npm, boilerplate)."
---

# Knowhow Init

## Tổng quan

Knowhow System là nghi thức tích luỹ tri thức có cấu trúc cho một kho (AI viết, người duyệt) qua 3 lớp:

| Lớp                    | Vai trò                                          |
| ---------------------- | ------------------------------------------------ |
| **Raw**                | Nguồn thô, immutable. Agent đọc nhưng không sửa. |
| **Wiki**               | Tri thức có cấu trúc, liên kết chéo.             |
| **Skills & Workflows** | Kiến thức thực thi, tái sử dụng.                 |


Phân biệt Skill và Workflow (bản rút gọn, định nghĩa chuẩn nằm trong `references/schema-template.md` mục "Phân biệt Skill và Workflow" mà Bước 3 sinh vào `SCHEMA.md`):

- **Skill** = thao tác khép kín, làm-theo-được, tái dùng cho task tương tự (được phép gắn domain).
- **Workflow** = chuỗi bước, gắn domain, gọi nhiều skill. Thay đổi khi đổi domain.

## Flow khởi tạo

### Bước 1: Thu thập thông tin

Hỏi user 3 câu:

1. **Tên kho tri thức** (dùng cho SCHEMA.md và log)
2. **Mô tả lĩnh vực** (1-2 câu, giải thích kho phục vụ việc gì)
3. **Kho có làm việc theo dự án/khách hàng/chiến dịch không?** (có/không). Nếu CÓ: seed thêm page type `project` (trang thực thể làm điểm neo cho tri thức cùng dự án, xem Bước 3 và 4). Nếu KHÔNG hoặc không chắc: bỏ qua, schema evolution sẽ thêm sau khi đủ tín hiệu.

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

**Nếu câu 3 = CÓ** (kho theo dự án): thêm dòng sau vào cuối bảng "Page Types" trong SCHEMA vừa sinh:

```
| project | wiki/project-<slug>.md | Trang thực thể dự án: mục tiêu, kết quả, neo tri thức cùng dự án |
```

Ghi kết quả vào `.knowhow/SCHEMA.md`.

### Bước 4: Sinh các file nội dung

**wiki/index.md**:

```markdown
# Wiki Index

## Decision

## Pattern

## Concept

## Troubleshooting

## Lesson
```

Nếu câu 3 = CÓ, thêm heading `## Project` vào cuối index.

**wiki/log.md** (thay `YYYY-MM-DD` bằng ngày hiện tại, `{{PROJECT_NAME}}` bằng tên kho):

```markdown
# Activity Log

## [YYYY-MM-DD] init | Khởi tạo .knowhow/ cho kho {{PROJECT_NAME}}
```

**skills/registry.md** (header phải khớp page-formats mục 6.1, gồm cột `Khi nào dùng` để `knowhow-run` match task sang skill):

```markdown
# Skill Registry

| Skill | Mô tả | Khi nào dùng | Version | Tags | Cập nhật |
|-------|--------|--------------|---------|------|----------|
```

**workflows/registry.md** (header phải khớp page-formats mục 6.2, gồm cột `Khi nào dùng` + `Tags` để `knowhow-run` match task sang workflow):

```markdown
# Workflow Registry

| Workflow | Mô tả | Khi nào dùng | Skills dùng | Version | Tags | Cập nhật |
|----------|--------|--------------|-------------|---------|------|----------|
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

Chỉ ghi vào **`AGENTS.md`** tại root workspace. Các config khác (`CLAUDE.md`, `GEMINI.md`) user tự xử lý.

Nội dung thêm nằm trong `references/agent-config-snippet.md`, đã bọc sẵn trong cặp marker:

```
<!-- knowhow:start -->
...
<!-- knowhow:end -->
```

Snippet dùng cú pháp `@` để agent tự nạp 4 file bản đồ vào context đầu phiên.

**Idempotent (3 nhánh)** — re-init phải **thay thế** nội dung trong marker để luôn đồng bộ phiên bản snippet mới nhất:

```bash
SNIPPET=references/agent-config-snippet.md
CONFIG=AGENTS.md

if [ ! -f "$CONFIG" ]; then
  # 1. Chưa có file → tạo mới + ghi block
  cp "$SNIPPET" "$CONFIG"
elif grep -q "knowhow:start" "$CONFIG"; then
  # 2. Đã có marker → thay thế toàn bộ vùng giữa start/end bằng snippet mới
  awk '
    /<!-- knowhow:start -->/ {while((getline line < "'"$SNIPPET"'")>0) print line; close("'"$SNIPPET"'"); skip=1; next}
    /<!-- knowhow:end -->/ {skip=0; next}
    !skip
  ' "$CONFIG" > "$CONFIG.tmp" && mv "$CONFIG.tmp" "$CONFIG"
else
  # 3. Có file, chưa marker → append block
  printf '\n' >> "$CONFIG"
  cat "$SNIPPET" >> "$CONFIG"
fi
```

### Bước 6: Ghi log hoàn tất

Xác nhận với user:

- Liệt kê các file đã tạo
- Nhắc user đọc `.knowhow/SCHEMA.md` để hiểu quy tắc vận hành
- Gợi ý bước tiếp theo: capture knowhow đầu tiên vào `inbox/`
