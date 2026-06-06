# Knowhow System: knowhow-run Spec v1.3

**Ngày**: 2026-06-06
**Phiên bản**: v1.3
**Quan hệ**: Bổ sung cho [design.md](2026-06-05-knowhow-system-design.md) v1.0, [review v1.1](2026-06-05-knowhow-system-review-v1.1.md), [schema-evolution v1.2](2026-06-05-knowhow-schema-evolution-design.md). File này KHÔNG thay thế spec gốc, chỉ chốt thiết kế skill mới `knowhow-run` và field `trigger` cho skill.
**Trạng thái**: thiết kế, chưa implement.

## La bàn

Thêm skill vận hành thứ 6 là `knowhow-run`: entrypoint chủ động để agent tra, load và làm theo skill/workflow đã đúc kết. Kèm một gia cố nhỏ ở khâu discovery là thêm field `trigger` cho skill. Skill này thuần markdown, không script, không phá tính agnostic của `.knowhow/`.

## Lõi vấn đề (đọc trước)

knowhow tích luỹ skill/workflow nhưng tiêu thụ chúng vẫn thụ động. Hai khâu cùng nghẽn:

- **Discovery**: agent biết có registry, nhưng skill registry chỉ có cột `Mô tả` một dòng, khó match đúng task sang skill. Workflow đã có `trigger` trong frontmatter, skill thì chưa, bất đối xứng.
- **Execution**: không có entrypoint chuẩn. Agent phải tự nhớ tra registry, tự mở file, tự làm. Dễ quên, dễ liếc một dòng mô tả rồi bịa thay vì đọc cả file.

Gốc chung: knowhow để mọi thứ cho agent tự giác, thiếu một entrypoint chủ động. Đây là phần `recall` cộng `execute` mà bộ nhớ dự án kiểu project-memory có, còn knowhow thiếu.

## Lõi giải pháp

Skill/workflow với wiki cùng là tri thức. Khác duy nhất: skill/workflow có thể hành động. Vì chúng chỉ là tri thức actionable dạng markdown, tiêu thụ chúng KHÔNG cần execution engine. Chỉ cần ba nhịp:

```
tra registry/index  →  load file bó khớp  →  đọc kỹ rồi làm theo
```

(bó = skill hoặc workflow đã đúc kết)

## Thiết kế knowhow-run

**Vai**: skill vận hành thứ 6. Chuyên tiêu thụ skill/workflow, không sản xuất.

**Trigger**:

- Agent tự gọi khi bắt đầu task thuộc domain dự án (qua dòng hướng dẫn trong `agent-config-snippet.md` và CLAUDE.md).
- User gọi tường minh khi muốn dùng một bó cụ thể.

**Input** (một trong ba):

- Tên skill/workflow cụ thể: load thẳng.
- Mô tả task: tra registry tìm bó khớp theo `trigger`, `Mô tả`, `tags`.
- Rỗng: liệt kê các bó khả dụng cho user chọn.

**Hành vi (ba nhịp)**:

1. **Tra**: đọc `skills/registry.md` và `workflows/registry.md`, match task với cột `Khi nào dùng` (field `trigger`), `Mô tả`, `tags`. Đầu phiên agent đã đọc `index` nên biết có bó nào.
2. **Load**: mở đúng file bó đã chọn, đọc HẾT nội dung. Không quyết định dựa trên một dòng registry.
3. **Làm theo**: thực hiện các bước trong file. Nếu là workflow gặp `→ Dùng skill: [[X]]`, load tiếp file skill con. Đệ quy tự nhiên, không cần orchestrator.

**Phân vai với knowhow-query**:

|          | `knowhow-query`         | `knowhow-run`          |
| -------- | ----------------------- | ---------------------- |
| Nguồn    | wiki                    | skills + workflows     |
| Bản chất | tri thức để biết        | tri thức để làm        |
| Output   | câu trả lời + trích dẫn | việc được làm theo bó  |

## Gia cố discovery: trigger cho skill

**Vấn đề**: skill registry chỉ có `Mô tả` một dòng, agent khó match. Workflow đã có `trigger` trong frontmatter, skill thì thiếu, bất đối xứng.

**Quyết định**: thêm field `trigger` vào skill frontmatter và cột `Khi nào dùng` vào skill registry.

Skill frontmatter (thêm `trigger`):

```yaml
---
type: skill
title: Tên skill
tags: []
trigger: Khi nào nên dùng skill này
input: Mô tả đầu vào
output: Mô tả đầu ra
reusable_across: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
---
```

Skill registry (thêm cột `Khi nào dùng`):

```markdown
| Skill | Mô tả | Khi nào dùng | Version | Tags | Cập nhật |
|-------|-------|--------------|---------|------|----------|
| [[parse-invoice]] | Parse PDF hóa đơn → JSON | Khi có hoá đơn PDF cần trích dữ liệu | 1.2 | pdf | 2026-06-20 |
```

- `knowhow-distill` bắt buộc điền `trigger` khi tạo hoặc sửa skill.
- `knowhow-lint rebuild-index` đọc thêm `trigger` khi sinh lại registry.

## Tối giản có chủ đích (đánh đổi đã chốt)

- **Không ghi log** mỗi lần chạy. Đánh đổi: lint không biết bó nào nóng hay chết, khó dọn bó chết sau này. Chấp nhận để giữ gọn. Nếu sau cần vòng phản hồi, thêm một dòng log là đủ.
- **Không checklist self-contained** nặng.
- **Không promote** bó thành skill vận hành thật. Bó vẫn là tài liệu trong `.knowhow/`.

## Không phá nguyên tắc nào

`knowhow-run` là skill vận hành, sống ở config agent như init, capture, distill, lint, query. Sản phẩm `.knowhow/` không đổi, vẫn markdown agnostic. Vì vậy việc thêm khả năng thực thi không làm `.knowhow/` mất tính agent-agnostic. Đây là lý do giữ run ở tầng vận hành, thay vì nhúng cơ chế chạy vào sản phẩm.

## File cần sửa khi implement

| File | Sửa gì |
|------|--------|
| `skills/knowhow-run/SKILL.md` | Tạo mới, mô tả ba nhịp tra → load → làm theo và cách match ba dạng input |
| `skills/knowhow-init/references/schema-template.md` | Thêm `trigger` vào schema skill; thêm cột `Khi nào dùng` vào mẫu skill registry |
| `skills/knowhow-capture/references/page-formats.md` | Thêm `trigger` vào format skill page |
| `skills/knowhow-distill/SKILL.md` | Bắt buộc điền `trigger` khi tạo hoặc sửa skill |
| `skills/knowhow-lint/SKILL.md` | `rebuild-index` đọc thêm `trigger` khi sinh registry |
| `skills/knowhow-init/references/agent-config-snippet.md` | Thêm dòng: khi đã tìm được skill/workflow, dùng `knowhow-run` để load và làm theo |
| `README.md` | Cập nhật 5 sang 6 skills, thêm `knowhow-run` vào bảng |

## Tiêu chí done v1.3

- [ ] `skills/knowhow-run/SKILL.md` tồn tại, mô tả rõ ba nhịp và ba dạng input.
- [ ] Skill có field `trigger`; skill registry có cột `Khi nào dùng`; `rebuild-index` giữ được.
- [ ] `knowhow-distill` điền `trigger` khi tạo hoặc sửa skill.
- [ ] `agent-config-snippet.md` có dòng gọi `knowhow-run`.
- [ ] README liệt kê 6 skills.
- [ ] run load đúng file và làm theo nội dung thật; workflow gọi skill con resolve được.
- [ ] run không ghi gì vào `.knowhow/` (tối giản thuần).
