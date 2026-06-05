# Knowhow System: Spec Thiết Kế

**Ngày**: 2026-06-05
**Phiên bản**: v1.0
**Mục tiêu**: Xây dựng bộ skills giúp quản lý knowhow, đúc kết skill/workflow từ quá trình làm việc, biến mỗi dự án thành kho tri thức tích luỹ có cấu trúc, AI viết và người duyệt.

## Tổng quan

### Vấn đề

Kiến thức tích luỹ trong quá trình làm việc (cách giải quyết vấn đề, quyết định kiến trúc, pattern hiệu quả, bài học từ lỗi) thường mất đi sau mỗi phiên trao đổi với AI. Không có cơ chế để:

- Ghi nhận knowhow một cách có hệ thống
- Đúc kết thành skill/workflow tái sử dụng
- Để AI Agent tự học từ knowhow đã tích luỹ

### Giải pháp

Hệ thống 3 lớp dữ liệu + 4 skills vận hành, cài vào bất kỳ dự án nào (code hoặc không code). Mỗi dự án tự chứa knowhow riêng trong thư mục `.knowhow/`, format markdown thuần, tương thích mọi AI Agent.

### Lấy cảm hứng từ

- **LLM Wiki** (Karpathy): Wiki là "persistent, compounding artifact". AI viết và duy trì, người dùng curate và hỏi đúng câu hỏi.
- **SOP Framework v2.0** (Minh Đỗ): Đóng gói quy trình ở mức chi tiết thao tác, có WHY, có ngoại lệ, có vòng phản hồi.

Knowhow System kết hợp cả hai: wiki layer cho tri thức + skill/workflow layer cho hành động.

---

## Kiến trúc

### 3 lớp dữ liệu

```
Raw (nguồn gốc, immutable)
  → Wiki (tri thức có cấu trúc, AI viết + người duyệt)
    → Skills & Workflows (kiến thức thực thi được, AI đề xuất + người duyệt)
```

| Lớp           | Ai viết                 | Mục đích                                        | Ví dụ                                                        |
| ------------- | ----------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| **Raw**       | Người / import          | Nguyên liệu thô, không sửa                      | Transcript trao đổi, ghi chú cuộc họp                        |
| **Wiki**      | AI viết, người duyệt    | Tri thức có cấu trúc, liên kết chéo             | "Retry pattern với Jitter", "Quyết định chuyển sang GraphQL" |
| **Skills**    | AI đề xuất, người duyệt | Công việc cụ thể, chạy độc lập, tái sử dụng cao | "Parse hóa đơn PDF", "Viết commit message chuẩn"             |
| **Workflows** | AI đề xuất, người duyệt | Chuỗi bước, gắn domain, gọi nhiều skill         | "Release checklist", "Onboard khách hàng"                    |


### Phân biệt Skill và Workflow

|                    | **Skill**                             | **Workflow**                                            |
| ------------------ | ------------------------------------- | ------------------------------------------------------- |
| **Bản chất**       | Một công việc cụ thể, khép kín        | Chuỗi bước để hoàn thành mục tiêu lớn hơn               |
| **Tái sử dụng**    | Cao, dùng được ở nhiều dự án/bối cảnh | Thấp hơn, thường gắn domain cụ thể                      |
| **Phụ thuộc**      | Chạy độc lập                          | Có thể gọi nhiều skill, có thứ tự và điều kiện rẽ nhánh |
| **Khi đổi domain** | Ít thay đổi                           | Thay đổi nhiều                                          |
| **Ví dụ**          | "Viết commit message chuẩn"           | "Release flow: lint → test → build → deploy → notify"   |


### Hai tầng hệ thống

| Tầng                                                       | Sống ở đâu                                | Agent-agnostic?                          | Vai trò                             |
| ---------------------------------------------------------- | ----------------------------------------- | ---------------------------------------- | ----------------------------------- |
| **Skills vận hành** (knowhow-init, capture, distill, lint) | Config agent (`~/.gemini/config/skills/`) | Không. Viết cho Antigravity, có thể port | Công cụ xây dựng và duy trì knowhow |
| **Sản phẩm knowhow** (`.knowhow/`)                         | Trong dự án, commit vào repo              | **Có**. Markdown thuần                   | Kiến thức dự án, mọi agent đọc được |


---

## Cấu trúc thư mục `.knowhow/`

```
.knowhow/
├── SCHEMA.md                  # Quy ước + hướng dẫn cho agent (đọc đầu tiên)
├── raw/                       # Lớp 1: nguồn thô, immutable
│   └── 2026-06-05-debug-session.md
├── inbox/                     # Bộ đệm: chờ đúc kết, chưa phân loại
│   └── 2026-06-05-pattern-retry.md
├── wiki/                      # Lớp 2: tri thức có cấu trúc
│   ├── index.md               # Mục lục toàn bộ wiki
│   ├── log.md                 # Nhật ký hoạt động (append-only)
│   ├── decisions/             # Quyết định + lý do + bối cảnh
│   ├── patterns/              # Pattern & anti-pattern đã chứng minh
│   ├── concepts/              # Thuật ngữ, khái niệm riêng dự án
│   └── troubleshooting/       # Sự cố đã gặp + cách xử lý
├── skills/                    # Lớp 3a: skill (chạy độc lập, tái sử dụng)
│   ├── registry.md            # Danh sách skill + metadata
│   └── parse-invoice.md
└── workflows/                 # Lớp 3b: workflow (nhiều bước, gọi skill)
    ├── registry.md            # Danh sách workflow + metadata
    └── release-checklist.md
```

---

## SCHEMA.md: File cấu hình dự án

```markdown
# Knowhow Schema

## Giới thiệu dự án
[Mô tả ngắn domain, mục đích dự án - sinh bởi knowhow-init]

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

## Naming Conventions

- File: `kebab-case.md`
- Skill: động từ + danh từ (`parse-invoice.md`, `write-commit-msg.md`)
- Workflow: danh từ mô tả quy trình (`release-checklist.md`, `customer-onboard.md`)
- Decision: mô tả quyết định (`rest-to-graphql.md`, `choose-postgres-over-mongo.md`)
- Pattern: mô tả pattern (`retry-with-jitter.md`, `cursor-pagination.md`)

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
```

---

## Format các loại page

### Wiki Page (decision / pattern / concept / troubleshooting)

```yaml
---
type: pattern | decision | concept | troubleshooting
title: Tên trang
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
---
```

Nội dung theo type:

**Decision**:

- Bối cảnh: tại sao phải quyết định
- Các lựa chọn đã xét: mỗi lựa chọn + ưu/nhược
- Quyết định: chọn gì
- Lý do: tại sao chọn
- Hệ quả: ảnh hưởng gì

**Pattern**:

- Bối cảnh: khi nào áp dụng
- Nội dung: cách làm chi tiết
- Ví dụ: từ dự án thực tế
- Anti-pattern: cách KHÔNG nên làm (nếu có)

**Concept**:

- Định nghĩa: nghĩa trong bối cảnh dự án này
- Liên quan: các concept liên quan
- Ví dụ: minh hoạ cụ thể

**Troubleshooting**:

- Triệu chứng: biểu hiện gì
- Nguyên nhân gốc: root cause
- Cách fix: từng bước
- Phòng ngừa: làm gì để không lặp lại

**Tất cả page đều có phần Changelog cuối**:

```markdown
## Changelog
- YYYY-MM-DD: Mô tả thay đổi
```

### Skill Page

```yaml
---
type: skill
title: Tên skill
tags: []
input: Mô tả đầu vào
output: Mô tả đầu ra
reusable_across: [bối cảnh 1, bối cảnh 2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
---
```

Nội dung:

- **Mục đích**: Skill này làm gì, khi nào dùng
- **Input / Output**: Rõ ràng, cụ thể
- **Các bước thực hiện**: Từng bước chi tiết ở mức thao tác
- **Edge cases**: Ngoại lệ và cách xử lý
- **Changelog**

### Workflow Page

```yaml
---
type: workflow
title: Tên workflow
tags: []
skills_used: []
trigger: Khi nào kích hoạt workflow này
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
---
```

Nội dung:

- **Khi nào dùng**: Trigger condition
- **Các bước**: Từng bước, mỗi bước có thể reference skill bằng `→ Dùng skill: [[skill-name]]`
- **Điều kiện rẽ nhánh**: Nếu X thì bước nào, nếu Y thì bước nào
- **Changelog**

### Registry Files

`**skills/registry.md**`:

```markdown
# Skill Registry

| Skill | Mô tả | Version | Tags | Cập nhật |
|-------|--------|---------|------|----------|
| [[parse-invoice]] | Parse PDF hóa đơn → JSON | 1.2 | pdf, data-extraction | 2026-06-20 |
| [[write-commit-msg]] | Viết commit message chuẩn | 1.0 | git, convention | 2026-06-05 |
```

`**workflows/registry.md**`:

```markdown
# Workflow Registry

| Workflow | Mô tả | Skills dùng | Version | Cập nhật |
|----------|--------|-------------|---------|----------|
| [[release-checklist]] | Quy trình release version mới | run-tests, deploy-staging | 2.0 | 2026-06-15 |
```

### Index và Log

`**wiki/index.md**`: Mục lục toàn bộ wiki, nhóm theo type:

```markdown
# Wiki Index

## Decisions
- [[rest-to-graphql]] - Chuyển từ REST sang GraphQL cho API v2

## Patterns
- [[retry-with-jitter]] - Retry API call với jitter thay exponential thuần
- [[cursor-pagination]] - Phân trang bằng cursor thay offset

## Concepts
- [[bounded-context]] - Ranh giới context trong kiến trúc dự án

## Troubleshooting
- [[n-plus-one-query]] - Lỗi N+1 khi join bảng orders
```

`**wiki/log.md**`: Nhật ký hoạt động, append-only, reverse chronological:

```markdown
# Activity Log

## 2026-06-12
- [distill] Cập nhật patterns/retry-with-jitter.md: thêm edge case timeout 429
- [distill] Tạo mới troubleshooting/n-plus-one-query.md

## 2026-06-05
- [init] Khởi tạo .knowhow/ cho dự án
- [capture] Ghi nhận 3 items vào inbox từ phiên debug API
```

---

## 4 Skills vận hành

### 1. `knowhow-init`

**Trigger**: Bắt đầu dự án mới, hoặc thêm `.knowhow/` vào dự án hiện có.

**Input**: Tên dự án, mô tả ngắn domain (hỏi user).

**Hành vi**:

1. Tạo toàn bộ cấu trúc thư mục `.knowhow/`
2. Sinh `SCHEMA.md` với quy ước mặc định, điền tên + mô tả dự án
3. Sinh `wiki/index.md`, `wiki/log.md` rỗng
4. Sinh `skills/registry.md`, `workflows/registry.md` rỗng
5. Thêm dòng hướng dẫn vào `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` (tuỳ file nào tồn tại, hoặc tạo mới)
6. Ghi log khởi tạo vào `wiki/log.md`

**Output**: Cấu trúc `.knowhow/` sẵn sàng sử dụng.

### 2. `knowhow-capture`

**Trigger**: User gọi sau phiên làm việc, hoặc khi muốn import nguồn ngoài, hoặc user nói "capture lại đi".

**Input**: 

- Cuộc trao đổi hiện tại (mặc định)
- Hoặc file đính kèm (transcript, ghi chú, tài liệu)

**Hành vi**:

1. Quét nguồn input, nhận diện knowhow tiềm năng:
  - Quyết định và lý do → candidate decision
  - Pattern/cách giải quyết mới → candidate pattern
  - Bài học, lỗi đã gặp → candidate troubleshooting
  - Khái niệm, thuật ngữ → candidate concept
  - Ý tưởng skill → candidate skill
  - Ý tưởng workflow → candidate workflow
2. Trình bày danh sách cho user, mỗi item gồm:
  - Loại (decision / pattern / skill / ...)
  - Tóm tắt 1-2 câu
  - Đề xuất phân loại
3. User duyệt: giữ / sửa / bỏ
4. Ghi các item được duyệt vào `inbox/` dưới dạng markdown
5. Nếu user cung cấp file nguồn, lưu vào `raw/`
6. Ghi log vào `wiki/log.md`

**Output**: Các file trong `inbox/` chờ distill.

> [!IMPORTANT]
>
> Capture chỉ ghi nhận vào inbox. KHÔNG ghi thẳng vào wiki, skills, hoặc workflows.

### 3. `knowhow-distill`

**Trigger**: User gọi khi inbox có nội dung, hoặc khi muốn đúc kết.

**Hành vi** (bán tự động):

1. Đọc `wiki/index.md` + `skills/registry.md` + `workflows/registry.md` (biết cái gì đã có)
2. Đọc các item trong `inbox/`
3. Với mỗi item, quyết định một trong các hành động:

| Tình huống                                      | Hành động                                   |
| ----------------------------------------------- | ------------------------------------------- |
| Chưa có gì liên quan                            | **Tạo mới** page                            |
| Đã có page liên quan, knowhow bổ sung thêm      | **Cập nhật** page cũ                        |
| Đã có page, knowhow thay thế cách cũ            | **Sửa** page cũ, đánh dấu cái cũ deprecated |
| Đã có skill/workflow, phát hiện bước thừa/thiếu | **Refine** skill/workflow                   |
| Nhiều page nhỏ cùng chủ đề                      | **Gộp** thành 1 page chất lượng hơn         |
| Không đáng lưu                                  | **Bỏ qua**                                  |


4. Trình bày đề xuất cho user, mỗi item gồm:
  - Item inbox nào
  - Hành động đề xuất (tạo mới / cập nhật / sửa / gộp / bỏ)
  - Page đích (nếu cập nhật/sửa)
  - Preview nội dung sẽ thay đổi
5. User duyệt từng item: đồng ý / sửa / bác bỏ
6. AI thực thi các item được duyệt:
  - Tạo/cập nhật page theo format chuẩn
  - Cập nhật `wiki/index.md` nếu thay đổi wiki
  - Cập nhật `skills/registry.md` hoặc `workflows/registry.md` nếu thay đổi skill/workflow
  - Thêm liên kết chéo giữa các page liên quan
  - Ghi changelog cuối mỗi page bị thay đổi
  - Ghi log vào `wiki/log.md`
7. Xoá item đã xử lý khỏi `inbox/`

**Output**: Wiki, skills, workflows được cập nhật. Inbox được dọn.

> [!IMPORTANT]
>
> Quy tắc cốt lõi: **ưu tiên cải tiến cái cũ hơn tạo mới**. Distill luôn kiểm tra cái đã có trước.

### 4. `knowhow-lint`

**Trigger**: User gọi định kỳ, hoặc khi knowhow đã tích luỹ nhiều.

**Hai chế độ**:

#### Quick Lint (mặc định)

Chạy nhanh, kiểm tra:

- Trang mồ côi (không có liên kết nào trỏ đến)
- Link `[[...]]` hỏng (reference page không tồn tại)
- Skill/workflow trong registry nhưng file không tồn tại (hoặc ngược lại)
- Page thiếu frontmatter bắt buộc
- Inbox tồn đọng lâu chưa xử lý

#### Consolidation (deep audit)

Rà soát sâu toàn bộ hệ thống:

- **Nhất quán nội dung**: Skill A nói "dùng cách X", wiki page B nói "dùng cách Y" → flagged
- **Nhất quán thuật ngữ**: Cùng khái niệm nhưng nhiều tên → đề xuất chuẩn hóa
- **Tính hợp lệ**: Skill/workflow reference file/tool/API không còn tồn tại trong dự án → flagged
- **Gộp & tách**: Nhiều page chồng chéo → đề xuất gộp. Page quá lớn → đề xuất tách
- **Độ phủ**: Lĩnh vực nào trong dự án chưa có knowhow? Đề xuất bổ sung
- **Chuỗi phụ thuộc**: Workflow gọi skill đã bị sửa → kiểm tra workflow còn đúng không
- **Lỗi thời**: Page lâu không cập nhật, confidence thấp → đề xuất archive hoặc review

**Output**: Báo cáo dạng checklist, nhóm theo loại vấn đề. User duyệt từng item.

```markdown
## Consolidation Report - YYYY-MM-DD

### Mâu thuẫn nội dung (N items)
- [ ] Mô tả mâu thuẫn + page liên quan + đề xuất xử lý

### Đề xuất gộp (N items)
- [ ] Page A + Page B có X% nội dung trùng → gộp?

### Lỗi thời (N items)
- [ ] Page X - lý do nghi ngờ lỗi thời → archive?

### Thiếu phủ (N items)
- [ ] Module/lĩnh vực Y chưa có knowhow

### Link hỏng (N items)
- [ ] Page Z reference [[abc]] nhưng abc không tồn tại
```

Sau khi user duyệt, AI thực thi các thay đổi và ghi log.

---

## Vòng đời tích luỹ tri thức

```
Lần 1: Gặp lỗi timeout
  → capture → distill → TẠO MỚI troubleshooting/api-timeout.md

Lần 2: Gặp lại, cách fix khác hiệu quả hơn
  → capture → distill → CẬP NHẬT api-timeout.md (thêm approach mới)

Lần 3: Nhận ra pattern chung cho resilience
  → capture → distill → TẠO MỚI skills/handle-api-resilience.md
                       + CẬP NHẬT api-timeout.md (link đến skill)

Lần 4: Kết hợp skill vào flow release
  → capture → distill → CẬP NHẬT workflows/release-checklist.md (thêm bước healthcheck)
                       + reference skill [[handle-api-resilience]]

Consolidation: Gộp 2 page trùng, chuẩn hóa thuật ngữ, archive cái lỗi thời
```

---

## Tích hợp đa agent

Khi `knowhow-init` chạy, nó thêm vào file cấu hình agent:

**CLAUDE.md** (Claude Code):

```markdown
## Knowhow
Đọc `.knowhow/SCHEMA.md` trước khi bắt đầu làm việc.
Khi gặp vấn đề, tra `.knowhow/wiki/index.md` trước.
Khi cần thực hiện quy trình, tra `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md`.
```

**AGENTS.md** (Codex) hoặc **GEMINI.md**: Nội dung tương tự.

Quy tắc: thêm vào file nào đã tồn tại. Nếu chưa có file nào, tạo `CLAUDE.md` mặc định.

---

## Phạm vi v1.0

### Có

- 4 skills: knowhow-init, knowhow-capture, knowhow-distill, knowhow-lint
- Cấu trúc `.knowhow/` hoàn chỉnh với SCHEMA.md
- Capture từ cuộc trao đổi AI + import file ngoài
- Distill bán tự động: AI đề xuất, user duyệt, ưu tiên cải tiến cái cũ
- Lint quick + consolidation (deep audit)
- Format markdown thuần, agent-agnostic
- Tích hợp CLAUDE.md / AGENTS.md / GEMINI.md
- Changelog mỗi page, version cho skill/workflow, activity log

### Không có (để sau)

- Search engine cho knowhow (dùng index.md + registry.md là đủ ở quy mô nhỏ)
- UI web để duyệt knowhow (dùng Obsidian hoặc đọc markdown trực tiếp)
- Auto-capture tự động sau mỗi phiên (v1.0 cần user gọi lệnh)
- Đồng bộ knowhow giữa nhiều dự án
- Template SCHEMA.md theo ngành/domain

---

## Thứ tự build

```
1. knowhow-init      ← nền tảng, build trước
2. knowhow-capture   ← cần init xong mới test được  
3. knowhow-distill   ← cần capture xong mới có inbox để test
4. knowhow-lint      ← cần distill xong mới có wiki để lint
```

---

## Kế hoạch kiểm chứng

### Test thực tế trên dự án thật

1. Chọn 1 dự án đang làm việc
2. Chạy `knowhow-init` → kiểm tra cấu trúc thư mục tạo đúng, SCHEMA.md đủ nội dung
3. Làm việc bình thường, gọi `knowhow-capture` → kiểm tra inbox có nội dung hợp lý
4. Gọi `knowhow-distill` → kiểm tra đề xuất đúng loại, format page đúng chuẩn
5. Capture thêm vài lần → kiểm tra distill ưu tiên cập nhật cái cũ, không tạo trùng
6. Gọi `knowhow-lint` quick → kiểm tra phát hiện link hỏng, orphan
7. Gọi `knowhow-lint` consolidation → kiểm tra phát hiện mâu thuẫn (cố tình tạo)
8. Mở dự án bằng agent khác (Claude Code) → kiểm tra agent tự đọc `.knowhow/SCHEMA.md` và tra cứu knowhow

### Tiêu chí thành công

- Agent mới mở dự án → tự biết có knowhow, tra cứu được
- Sau 10 lần capture + distill → knowhow tích luỹ, không trùng lặp, có cross-reference
- Distill phát hiện page cũ liên quan và đề xuất cập nhật thay vì tạo mới
- Consolidation phát hiện được mâu thuẫn nếu cố tình tạo
- Format page nhất quán, frontmatter đúng schema
