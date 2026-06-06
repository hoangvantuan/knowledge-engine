# Knowhow Run (v1.3) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Thêm skill vận hành thứ 6 `knowhow-run` — entrypoint chủ động để agent tra, load và làm theo skill/workflow đã đúc kết — kèm gia cố discovery: thêm field `trigger` cho skill (đối xứng với workflow).

**Architecture:** `knowhow-run` thuần markdown, không script, sống ở config agent như 5 skill kia. Nó tiêu thụ bó (skill/workflow trong `.knowhow/`) qua ba nhịp `tra registry → load file → đọc hết rồi làm theo`. Để khâu "tra" match đúng, skill được bổ sung field `trigger` trong frontmatter và cột `Khi nào dùng` trong registry, do `knowhow-distill` điền và `knowhow-lint rebuild-index` sinh lại. Sản phẩm `.knowhow/` không đổi, vẫn markdown agnostic; run KHÔNG ghi gì vào `.knowhow/`.

**Tech Stack:** Markdown thuần. Mỗi "skill vận hành" là một file `SKILL.md` chứa chỉ dẫn cho AI agent đọc rồi làm theo. KHÔNG có code biên dịch, KHÔNG có test runner. "Verify" = kiểm phần cơ học bằng `grep`/`ls`/`test` và đọc lại checklist cho phần phán đoán.

---

## Bối cảnh cho người thực thi (đọc trước khi bắt đầu)

Bạn là dev giỏi nhưng chưa biết hệ này. Vài điều cốt lõi phải nắm:

1. **Hai loại "skill", đừng nhầm.**
   - **Skill vận hành**: file `skills/knowhow-*/SKILL.md` trong repo này. Frontmatter dùng `name` + `description` (xem [knowhow-query/SKILL.md](../../../skills/knowhow-query/SKILL.md)). Đây là công cụ AI đọc để xây/duy trì/tiêu thụ knowhow. `knowhow-run` là skill vận hành thứ 6.
   - **Skill sản phẩm**: file `.knowhow/skills/<slug>.md` sinh ra trong dự án người dùng. Frontmatter dùng `type: skill`, `title`, `tags`, `input`, `output`... Field `trigger` mới thêm là cho LOẠI NÀY, không phải cho `knowhow-run`.

2. **Sửa skill = sửa văn bản chỉ dẫn**, không sửa code. Mọi file là `.md`.

3. **Bất biến không được phá:** mọi knowhow qua `inbox/` trước; `knowhow-run` KHÔNG ghi gì vào `.knowhow/` (kể cả `wiki/log.md`). Đây là đánh đổi đã chốt ở spec v1.3 (mục "Tối giản có chủ đích").

4. **Phân vai run vs query:** `knowhow-query` = tri thức để *biết* (nguồn: wiki). `knowhow-run` = tri thức để *làm* (nguồn: skills + workflows).

5. **Điểm lệch spec ↔ codebase (đã đối chiếu thực tế):** Spec "File cần sửa" dòng 109 ghi sửa `schema-template.md` để thêm `trigger` vào schema skill + cột `Khi nào dùng` vào skill registry. Nhưng `schema-template.md` thực tế CHỈ có bảng Page Types, KHÔNG chứa skill frontmatter chi tiết hay skill registry mẫu. Hai thứ đó nằm ở [page-formats.md](../../../skills/knowhow-capture/references/page-formats.md) (mục 3 "Skill Page Format" và mục 6.1 "Skill Registry"). Vì vậy Task 2 và Task 3 sửa `page-formats.md` (nguồn chân lý thật), KHÔNG sửa `schema-template.md`.

### File sẽ tạo / sửa (bản đồ trước khi vào việc)

**Tạo mới:**
- `skills/knowhow-run/SKILL.md` — skill vận hành thứ 6: ba nhịp tra → load → làm theo, ba dạng input, phân vai với query, quy tắc cứng "không ghi vào .knowhow/".

**Sửa:**
- `skills/knowhow-capture/references/page-formats.md` — mục 3: thêm `trigger` vào skill frontmatter + ví dụ; mục 6.1: thêm cột `Khi nào dùng` vào skill registry.
- `skills/knowhow-distill/SKILL.md` — bắt buộc điền `trigger` khi tạo/sửa skill sản phẩm.
- `skills/knowhow-lint/SKILL.md` — rebuild-index đọc thêm `trigger` + sinh cột `Khi nào dùng`; frontmatter check 1c thêm `trigger` cho skill (đối xứng workflow).
- `skills/knowhow-init/references/agent-config-snippet.md` — thêm dòng: tìm được bó thì dùng `knowhow-run` để load và làm theo.
- `README.md` — 5 → 6 skills: số đếm, bảng skills, note, cấu trúc kho mã.

### Thứ tự thực thi

Task 1 (file mới, độc lập) → Task 2, 3 (sửa page-formats, định nghĩa schema trước) → Task 4, 5 (distill/lint dựa vào schema mới) → Task 6 (config snippet) → Task 7 (README). Mỗi task commit riêng.

---

## Task 1: Tạo skill vận hành `knowhow-run`

Tạo file `SKILL.md` mô tả đầy đủ ba nhịp và ba dạng input. Đây là file lớn nhất và độc lập, không phụ thuộc task khác.

**Files:**
- Create: `skills/knowhow-run/SKILL.md`

- [ ] **Step 1: Tạo thư mục skill**

Run:
```bash
mkdir -p skills/knowhow-run
```

- [ ] **Step 2: Viết toàn bộ `skills/knowhow-run/SKILL.md`**

Tạo file `skills/knowhow-run/SKILL.md` với nội dung NGUYÊN VĂN sau:

````markdown
---
name: knowhow-run
description: "Entrypoint chủ động để tiêu thụ skill/workflow đã đúc kết trong .knowhow/. Ba nhịp: tra registry → load file bó khớp → đọc hết rồi làm theo. Nhận 3 dạng input: tên bó cụ thể (load thẳng), mô tả task (tra registry theo trigger/Mô tả/tags), hoặc rỗng (liệt kê bó khả dụng). Workflow gặp skill con resolve đệ quy. KHÔNG sản xuất, KHÔNG ghi vào .knowhow/. Trigger: 'knowhow run', 'chạy skill X', 'làm theo workflow Y', 'dùng knowhow để làm', khi bắt đầu task thuộc domain dự án và cần dùng bó đã tích luỹ."
---

# Knowhow Run

Entrypoint chủ động để tra, load và làm theo skill/workflow đã đúc kết. Đây là phần `recall` + `execute` mà knowhow trước đây thiếu: hệ tích luỹ bó tốt, nhưng tiêu thụ chúng vẫn thụ động (agent phải tự nhớ tra registry, tự mở file, dễ liếc một dòng mô tả rồi bịa).

Lõi: skill/workflow cùng là tri thức như wiki, khác duy nhất là chúng *hành động được*. Vì chỉ là tri thức actionable dạng markdown, tiêu thụ KHÔNG cần execution engine. Chỉ cần ba nhịp:

```
tra registry/index  →  load file bó khớp  →  đọc kỹ rồi làm theo
```

(bó = một skill hoặc một workflow đã đúc kết trong .knowhow/)

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user chạy `knowhow-init` trước.

## Phạm vi agent

`knowhow-run` thuần đọc-file-rồi-làm-theo, không thao tác đặc thù agent nào. Vì là markdown thuần, nó ít phụ thuộc agent hơn 5 skill kia. Dù vậy nó vẫn là skill vận hành, sống ở config agent, không nhúng vào sản phẩm `.knowhow/`.

## Phân vai với knowhow-query

|          | knowhow-query           | knowhow-run            |
| -------- | ----------------------- | ---------------------- |
| Nguồn    | wiki                    | skills + workflows     |
| Bản chất | tri thức để biết        | tri thức để làm        |
| Output   | câu trả lời + trích dẫn | việc được làm theo bó  |

Task "cần biết/tra cứu" → dùng `knowhow-query`. Task "cần làm theo một quy trình đã đúc kết" → dùng skill này.

## Ba dạng input

Xác định input thuộc dạng nào rồi rẽ nhánh:

| Dạng input | Ví dụ | Nhịp bắt đầu |
|---|---|---|
| Tên bó cụ thể | "chạy skill parse-invoice", "làm theo workflow release-flow" | Bỏ qua Tra, vào thẳng Load |
| Mô tả task | "tôi có file PDF hoá đơn cần trích dữ liệu" | Bắt đầu từ Tra |
| Rỗng | "knowhow run" (không kèm gì) | Liệt kê bó khả dụng cho user chọn |

## Ba nhịp

### Nhịp 1: Tra (chỉ khi input là mô tả task hoặc rỗng)

1. Đọc `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md`. (Đầu phiên agent đã đọc `SCHEMA.md` + index nên biết có bó nào.)
2. Nếu input RỖNG: liệt kê các bó khả dụng (tên + cột `Khi nào dùng` + `Mô tả`) cho user chọn, rồi dừng chờ chọn.
3. Nếu input là MÔ TẢ TASK: match task với các bó. Thứ tự ưu tiên khi match:
   - Cột `Khi nào dùng` (field `trigger`) — tín hiệu mạnh nhất, mô tả đúng tình huống kích hoạt.
   - Cột `Mô tả`.
   - Cột `Tags`.
   Rút 3-5 từ khoá từ task, đối chiếu. Nếu registry chưa đủ để chắc, grep nội dung thật:
   ```bash
   grep -ril "<từ khoá>" .knowhow/skills .knowhow/workflows
   ```
4. Chọn bó khớp nhất:
   - Đúng 1 bó khớp rõ → sang Nhịp 2.
   - Nhiều bó khớp → trình bày danh sách ngắn (tên + `Khi nào dùng`), hỏi user chọn.
   - Không bó nào khớp → nói thẳng "chưa có skill/workflow cho việc này", gợi ý `knowhow-capture` + `knowhow-distill` để đúc kết sau. KHÔNG bịa một bó.

### Nhịp 2: Load

1. Mở đúng file bó đã chọn:
   - Skill: `.knowhow/skills/<slug>.md`
   - Workflow: `.knowhow/workflows/<slug>.md`
   - Nếu type đã tách subfolder (schema tiến hoá), tìm đệ quy:
     ```bash
     find .knowhow/skills .knowhow/workflows -name "<slug>.md"
     ```
2. Đọc HẾT nội dung file. KHÔNG quyết định dựa trên một dòng registry. Một dòng mô tả không đủ để làm đúng; phải đọc cả các bước, edge case, điều kiện rẽ nhánh.

### Nhịp 3: Làm theo

1. Thực hiện lần lượt các bước ghi trong file bó. Bám sát nội dung thật, không tự chế thêm bước.
2. Nếu là **workflow** và gặp `→ Dùng skill: [[X]]`:
   - Resolve `[[X]]` về file skill con (`.knowhow/skills/X.md`, hoặc `find` nếu subfolder).
   - Quay lại Nhịp 2 (Load) cho skill con: đọc hết rồi làm theo.
   - Đệ quy tự nhiên, KHÔNG cần orchestrator. Làm xong skill con thì quay lại bước kế của workflow.
3. Tôn trọng "Điều kiện rẽ nhánh" của workflow: theo đúng nhánh, không skip bước bị chặn.
4. Nếu bó tham chiếu wiki page `[[slug]]` để lấy bối cảnh, đọc page đó khi cần (như knowhow-query làm), nhưng trọng tâm vẫn là làm theo bó.

## Quy tắc cứng

1. **KHÔNG ghi gì vào `.knowhow/`.** run thuần tiêu thụ: không tạo inbox item, không sửa wiki/skill/workflow, KHÔNG ghi `wiki/log.md`. (Đánh đổi đã chốt spec v1.3: bỏ log để giữ gọn. Nếu sau cần vòng phản hồi bó-nóng/bó-chết, thêm một dòng log là đủ.)
2. **Đọc hết file bó trước khi làm.** Cấm liếc một dòng registry rồi bịa các bước.
3. **Không bịa bó.** Không có bó khớp thì nói không có, gợi ý đúc kết. Không tự nghĩ ra quy trình rồi gán cho một bó không tồn tại.
4. **Không sản xuất.** Nếu trong lúc làm phát hiện bó thiếu/sai bước, KHÔNG tự sửa bó ở đây. Ghi nhận và gợi ý user chạy `knowhow-distill` để refine. (Tách tiêu thụ khỏi sản xuất.)
5. **Tôn trọng cửa duy nhất.** Bài học mới sinh trong lúc chạy (nếu đáng lưu) đi qua `knowhow-capture` → inbox, KHÔNG ghi thẳng.

## Edge cases

- **Bó có `status: deprecated`**: vẫn load được nhưng cảnh báo "bó này deprecated, có thể có cách mới hơn", hỏi user có tiếp không.
- **Bó có `status: archived`**: nằm trong `archive/`, KHÔNG nên chạy. Báo user bó đã lỗi thời.
- **Workflow trỏ skill con không resolve** (`[[X]]` không tìm thấy file): dừng nhịp đó, báo link hỏng, gợi ý chạy `knowhow-lint quick` để bắt link hỏng.
- **Registry rỗng** (chưa có bó nào): báo "dự án chưa đúc kết skill/workflow nào", gợi ý `knowhow-capture` + `knowhow-distill`.
````

- [ ] **Step 3: Verify cấu trúc và nội dung file**

Run:
```bash
test -f skills/knowhow-run/SKILL.md && echo "FILE OK"
grep -c "Nhịp 1: Tra\|Nhịp 2: Load\|Nhịp 3: Làm theo" skills/knowhow-run/SKILL.md
grep -q "name: knowhow-run" skills/knowhow-run/SKILL.md && echo "FRONTMATTER OK"
grep -q "KHÔNG ghi gì vào" skills/knowhow-run/SKILL.md && echo "NO-WRITE RULE OK"
grep -q "Tên bó cụ thể\|Mô tả task\|Rỗng" skills/knowhow-run/SKILL.md && echo "3 INPUTS OK"
```
Expected:
- `FILE OK`
- `3` (đủ ba heading nhịp)
- `FRONTMATTER OK`
- `NO-WRITE RULE OK`
- `3 INPUTS OK`

- [ ] **Step 4: Đọc lại checklist phán đoán**

Đọc lại file vừa tạo, xác nhận đủ các điểm spec yêu cầu (mục "Tiêu chí done v1.3"):
- [ ] Mô tả rõ ba nhịp tra → load → làm theo.
- [ ] Mô tả rõ ba dạng input (tên bó / mô tả task / rỗng) và nhịp bắt đầu của mỗi dạng.
- [ ] Có bảng phân vai với `knowhow-query`.
- [ ] Workflow gặp `→ Dùng skill: [[X]]` resolve đệ quy được (Nhịp 3 mục 2).
- [ ] Quy tắc cứng số 1: run KHÔNG ghi gì vào `.knowhow/`.

- [ ] **Step 5: Commit**

```bash
git add skills/knowhow-run/SKILL.md
git commit -m "feat(v1.3): thêm skill vận hành knowhow-run (ba nhịp tra/load/làm theo)"
```

---

## Task 2: Thêm `trigger` vào skill frontmatter (page-formats mục 3)

Skill sản phẩm cần field `trigger` trong frontmatter để khâu "tra" của run match đúng task. Đối xứng với workflow đã có `trigger`. Sửa cả phần frontmatter mẫu lẫn ví dụ.

**Files:**
- Modify: `skills/knowhow-capture/references/page-formats.md` (mục 3, quanh dòng 428-441 và ví dụ 475-486)

- [ ] **Step 1: Thêm `trigger` vào frontmatter mẫu của Skill Page Format**

Mở [page-formats.md](../../../skills/knowhow-capture/references/page-formats.md). Tại mục `## 3. Skill Page Format` → `### Frontmatter`, khối hiện tại là:

```yaml
---
type: skill
title: "Tên skill"
tags: []
input: "Mô tả đầu vào"
output: "Mô tả đầu ra"
reusable_across: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
status: active   # active | deprecated | archived
---
```

Sửa thành (thêm dòng `trigger` ngay sau `tags`, đặt cạnh metadata mô tả ngữ cảnh):

```yaml
---
type: skill
title: "Tên skill"
tags: []
trigger: "Khi nào nên dùng skill này"
input: "Mô tả đầu vào"
output: "Mô tả đầu ra"
reusable_across: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
status: active   # active | deprecated | archived
---
```

- [ ] **Step 2: Thêm `trigger` vào frontmatter của ví dụ parse-invoice**

Tại `### Ví dụ` của mục 3, khối frontmatter ví dụ hiện tại là:

```yaml
---
type: skill
title: "Parse PDF hóa đơn thành structured JSON"
tags: [pdf, parsing, invoice]
input: "File PDF hóa đơn (VAT Việt Nam)"
output: "JSON object chứa seller, buyer, items, totals"
reusable_across: [accounting-module, report-generator]
created: 2026-06-05
updated: 2026-06-05
version: "1.0"
status: active
---
```

Sửa thành (thêm dòng `trigger` ngay sau `tags`):

```yaml
---
type: skill
title: "Parse PDF hóa đơn thành structured JSON"
tags: [pdf, parsing, invoice]
trigger: "Khi có hoá đơn PDF (VAT Việt Nam) cần trích dữ liệu thành JSON"
input: "File PDF hóa đơn (VAT Việt Nam)"
output: "JSON object chứa seller, buyer, items, totals"
reusable_across: [accounting-module, report-generator]
created: 2026-06-05
updated: 2026-06-05
version: "1.0"
status: active
---
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -c 'trigger:' skills/knowhow-capture/references/page-formats.md
```
Expected: số đếm tăng đúng 2 so với trước (mục 3 frontmatter + ví dụ mục 3). Workflow mục 4 đã có sẵn 2 dòng `trigger:` từ trước, nên tổng cuối cùng là `4`.

Run kiểm vị trí (trigger phải nằm trong mục 3, trước mục 4):
```bash
grep -n 'trigger:\|## 3. Skill Page Format\|## 4. Workflow Page Format' skills/knowhow-capture/references/page-formats.md
```
Expected: có ≥1 dòng `trigger:` nằm giữa heading mục 3 và heading mục 4.

- [ ] **Step 4: Commit**

```bash
git add skills/knowhow-capture/references/page-formats.md
git commit -m "feat(v1.3): thêm field trigger vào skill page format"
```

---

## Task 3: Thêm cột `Khi nào dùng` vào skill registry (page-formats mục 6.1)

Skill registry cần cột `Khi nào dùng` (= field `trigger`) để run match task sang bó. Đối xứng: registry trước đây chỉ có `Mô tả` một dòng, agent khó match.

**Files:**
- Modify: `skills/knowhow-capture/references/page-formats.md` (mục 6.1, quanh dòng 651-657)

- [ ] **Step 1: Thêm cột `Khi nào dùng` vào mẫu bảng Skill Registry**

Tại `### 6.1. Skill Registry`, khối markdown hiện tại là:

```markdown
# Skill Registry

| Skill | Mô tả | Version | Tags | Cập nhật |
|-------|--------|---------|------|----------|
| [[slug]] | mô tả ngắn | 1.0 | tag1, tag2 | YYYY-MM-DD |
```

Sửa thành (chèn cột `Khi nào dùng` ngay sau `Mô tả`):

```markdown
# Skill Registry

| Skill | Mô tả | Khi nào dùng | Version | Tags | Cập nhật |
|-------|--------|--------------|---------|------|----------|
| [[slug]] | mô tả ngắn | khi nào nên dùng (= trigger) | 1.0 | tag1, tag2 | YYYY-MM-DD |
```

- [ ] **Step 2: Verify**

Run:
```bash
grep -n 'Khi nào dùng' skills/knowhow-capture/references/page-formats.md
```
Expected: 1 dòng hit, nằm trong mục `### 6.1. Skill Registry` (header bảng).

Kiểm bảng vẫn cân cột (header 6 cột, separator 6 cột):
```bash
grep -A2 '# Skill Registry' skills/knowhow-capture/references/page-formats.md | grep -c '|'
```
Expected: `3` (header + separator + 1 dòng ví dụ, mỗi dòng có dấu `|`).

- [ ] **Step 3: Commit**

```bash
git add skills/knowhow-capture/references/page-formats.md
git commit -m "feat(v1.3): thêm cột Khi nào dùng vào skill registry"
```

---

## Task 4: `knowhow-distill` bắt buộc điền `trigger` khi tạo/sửa skill

distill là nơi sinh/sửa skill sản phẩm. Phải bắt buộc điền `trigger`, nếu không registry sẽ thiếu cột `Khi nào dùng` và run mất tín hiệu match mạnh nhất.

**Files:**
- Modify: `skills/knowhow-distill/SKILL.md` (Bước 5 "Thực thi", mục 1, quanh dòng 114-116)

- [ ] **Step 1: Thêm yêu cầu điền `trigger` vào Bước 5 mục 1**

Mở [knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md). Tại `### Bước 5: Thực thi`, mục 1 hiện tại là:

```markdown
1. **Tạo hoặc cập nhật page** theo format chuẩn.
   - Format reference: `../knowhow-capture/references/page-formats.md` (đường dẫn tương đối từ thư mục skill này). Nếu không resolve được, dùng format frontmatter cơ bản bên dưới.
   - Format frontmatter cơ bản: title, type, tags, created, updated.
```

Sửa thành (thêm một gạch đầu dòng bắt buộc `trigger` cho skill):

```markdown
1. **Tạo hoặc cập nhật page** theo format chuẩn.
   - Format reference: `../knowhow-capture/references/page-formats.md` (đường dẫn tương đối từ thư mục skill này). Nếu không resolve được, dùng format frontmatter cơ bản bên dưới.
   - Format frontmatter cơ bản: title, type, tags, created, updated.
   - **Skill: BẮT BUỘC điền `trigger`** trong frontmatter (mô tả "khi nào nên dùng skill này"). Đây là tín hiệu mạnh nhất để `knowhow-run` match task sang skill; thiếu nó registry sẽ trống cột `Khi nào dùng`. Đối xứng với workflow (vốn đã bắt buộc `trigger`). Khi REFINE skill cũ chưa có `trigger`, bổ sung luôn trong lần sửa này.
```

- [ ] **Step 2: Verify**

Run:
```bash
grep -n 'BẮT BUỘC điền `trigger`' skills/knowhow-distill/SKILL.md
```
Expected: 1 dòng hit, nằm trong `### Bước 5: Thực thi`.

- [ ] **Step 3: Commit**

```bash
git add skills/knowhow-distill/SKILL.md
git commit -m "feat(v1.3): distill bắt buộc điền trigger khi tạo/sửa skill"
```

---

## Task 5: `knowhow-lint` đọc `trigger` (rebuild-index + frontmatter check)

Hai chỗ trong lint cần biết `trigger`: (a) rebuild-index sinh lại registry phải đọc `trigger` và sinh cột `Khi nào dùng`; (b) frontmatter check 1c nên kiểm skill có `trigger` (đối xứng workflow), để bắt skill cũ thiếu trigger.

**Files:**
- Modify: `skills/knowhow-lint/SKILL.md` (dòng 170: rebuild-index skill; dòng 57-59: frontmatter check 1c)

- [ ] **Step 1: rebuild-index đọc thêm `trigger` khi sinh skill registry**

Mở [knowhow-lint/SKILL.md](../../../skills/knowhow-lint/SKILL.md). Tại `## Chế độ 3: Rebuild Index` → `### Bước thực hiện`, mục 3 hiện tại là:

```markdown
3. Quét frontmatter mọi file trong `skills/` (đệ quy, `find .knowhow/skills -name "*.md" -not -name "registry.md"`) (trừ registry.md): đọc `title`, `version`, `tags`, `updated`. Sinh `skills/registry.md` theo format page-formats mục 6.1, sort alphabet.
```

Sửa thành (thêm `trigger` vào danh sách field đọc, nêu rõ map sang cột `Khi nào dùng`):

```markdown
3. Quét frontmatter mọi file trong `skills/` (đệ quy, `find .knowhow/skills -name "*.md" -not -name "registry.md"`) (trừ registry.md): đọc `title`, `trigger`, `version`, `tags`, `updated`. Sinh `skills/registry.md` theo format page-formats mục 6.1 (cột `Khi nào dùng` lấy từ `trigger`; skill cũ thiếu `trigger` → để ô trống và sẽ bị frontmatter check 1c báo), sort alphabet.
```

- [ ] **Step 2: Thêm `trigger` vào frontmatter check 1c cho skill**

Tại `### 1c. Frontmatter check`, khối hiện tại là:

```markdown
Thêm theo loại:
- **Skill**: `version`, `input`, `output`
- **Workflow**: `version`, `trigger`
- **Wiki (mọi type ∈ TYPES)**: `confidence`
```

Sửa dòng Skill thành (thêm `trigger`):

```markdown
Thêm theo loại:
- **Skill**: `version`, `trigger`, `input`, `output`
- **Workflow**: `version`, `trigger`
- **Wiki (mọi type ∈ TYPES)**: `confidence`
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n "đọc \`title\`, \`trigger\`" skills/knowhow-lint/SKILL.md
grep -n '\*\*Skill\*\*: `version`, `trigger`, `input`, `output`' skills/knowhow-lint/SKILL.md
```
Expected: mỗi lệnh ra đúng 1 dòng hit.

- [ ] **Step 4: Commit**

```bash
git add skills/knowhow-lint/SKILL.md
git commit -m "feat(v1.3): lint đọc trigger ở rebuild-index + frontmatter check"
```

---

## Task 6: `agent-config-snippet.md` gọi `knowhow-run`

Snippet này được `knowhow-init` chèn vào `CLAUDE.md`/`AGENTS.md`/`GEMINI.md` của dự án. Phải có dòng: tìm được bó thì dùng `knowhow-run` để load và làm theo. Đây là điểm để agent tự gọi run khi vào task domain.

**Files:**
- Modify: `skills/knowhow-init/references/agent-config-snippet.md` (mục danh sách hướng dẫn, dòng 5-9)

- [ ] **Step 1: Thêm dòng gọi `knowhow-run`**

Mở [agent-config-snippet.md](../../../skills/knowhow-init/references/agent-config-snippet.md). Danh sách hiện tại là:

```markdown
1. Đọc `.knowhow/SCHEMA.md` trước khi bắt đầu làm việc
2. Khi gặp vấn đề, tra `.knowhow/wiki/index.md` trước
3. Khi cần thực hiện quy trình, tra `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md`
4. Sau phiên làm việc có bài học đáng ghi nhận, đề xuất capture vào `.knowhow/inbox/`
5. Khi `.knowhow/inbox/` có ≥ 5 item hoặc có item cũ hơn 7 ngày, chủ động nhắc user chạy `knowhow-distill` để đúc kết, tránh inbox tồn đọng.
```

Sửa mục 3 và chèn mục mới ngay sau (để thành chuỗi "tra → dùng run"), đánh số lại:

```markdown
1. Đọc `.knowhow/SCHEMA.md` trước khi bắt đầu làm việc
2. Khi gặp vấn đề, tra `.knowhow/wiki/index.md` trước
3. Khi cần thực hiện quy trình, tra `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md` (match task theo cột `Khi nào dùng`, `Mô tả`, `Tags`)
4. Khi đã tìm được skill/workflow phù hợp, dùng `knowhow-run` để load file bó và làm theo, KHÔNG chỉ liếc một dòng mô tả rồi tự làm
5. Sau phiên làm việc có bài học đáng ghi nhận, đề xuất capture vào `.knowhow/inbox/`
6. Khi `.knowhow/inbox/` có ≥ 5 item hoặc có item cũ hơn 7 ngày, chủ động nhắc user chạy `knowhow-distill` để đúc kết, tránh inbox tồn đọng.
```

- [ ] **Step 2: Verify**

Run:
```bash
grep -n 'knowhow-run' skills/knowhow-init/references/agent-config-snippet.md
grep -c '^[0-9]\.' skills/knowhow-init/references/agent-config-snippet.md
```
Expected:
- 1 dòng hit chứa `knowhow-run` (mục 4).
- `6` (danh sách đánh số lại từ 1 đến 6, liền mạch).

- [ ] **Step 3: Commit**

```bash
git add skills/knowhow-init/references/agent-config-snippet.md
git commit -m "feat(v1.3): agent-config-snippet hướng dẫn dùng knowhow-run"
```

---

## Task 7: README cập nhật 5 → 6 skills

README đang nói "5 skills vận hành" ở nhiều chỗ. Cập nhật số đếm, bảng skills (thêm `knowhow-run`), note, và cấu trúc kho mã.

**Files:**
- Modify: `README.md` (dòng 19, 60, bảng 62-68, note 95, cấu trúc 140-145)

- [ ] **Step 1: Sửa câu mô tả giải pháp (dòng 19)**

Mở [README.md](../../../README.md). Dòng hiện tại:

```markdown
Một hệ thống **3 lớp dữ liệu** + **5 skills vận hành**, cài vào bất kỳ dự án nào (code hoặc không code). Knowhow đi qua một cửa duy nhất là `inbox/`, rồi được đúc kết thành tri thức có cấu trúc, được rà soát định kỳ, và tự tiến hoá khuôn khi dự án lớn lên.
```

Sửa `5 skills vận hành` → `6 skills vận hành`:

```markdown
Một hệ thống **3 lớp dữ liệu** + **6 skills vận hành**, cài vào bất kỳ dự án nào (code hoặc không code). Knowhow đi qua một cửa duy nhất là `inbox/`, rồi được đúc kết thành tri thức có cấu trúc, được rà soát định kỳ, và tự tiến hoá khuôn khi dự án lớn lên.
```

- [ ] **Step 2: Sửa heading section (dòng 60)**

Dòng hiện tại:

```markdown
## 5 skills vận hành
```

Sửa thành:

```markdown
## 6 skills vận hành
```

- [ ] **Step 3: Thêm hàng `knowhow-run` vào bảng skills**

Bảng hiện kết thúc ở hàng `knowhow-query` (dòng 68):

```markdown
| **knowhow-query** | Trả lời câu hỏi từ knowhow, trích dẫn `[[slug]]`, phát tín hiệu khi không trúng page | Khi cần tra cứu tri thức đã tích luỹ |
```

Thêm NGAY SAU hàng đó một hàng mới:

```markdown
| **knowhow-run** | Tiêu thụ skill/workflow đã đúc kết: tra registry → load file bó → làm theo. Không ghi vào `.knowhow/` | Khi bắt đầu task domain cần làm theo một bó đã có |
```

- [ ] **Step 4: Sửa note "5 skill vận hành" (dòng 95)**

Dòng hiện tại (trong block `> [!NOTE]`):

```markdown
> 5 skill vận hành hiện được viết cho **Antigravity (Gemini)**. Sản phẩm knowhow (`.knowhow/`) là markdown thuần nên mọi agent (Claude Code, Codex, ...) đều **đọc** được, nhưng để **chạy** capture/distill/lint/query trên agent khác cần port skill trước.
```

Sửa `5 skill` → `6 skill` và bổ sung `run` vào danh sách chạy:

```markdown
> 6 skill vận hành hiện được viết cho **Antigravity (Gemini)**. Sản phẩm knowhow (`.knowhow/`) là markdown thuần nên mọi agent (Claude Code, Codex, ...) đều **đọc** được, nhưng để **chạy** capture/distill/lint/query/run trên agent khác cần port skill trước.
```

- [ ] **Step 5: Thêm `knowhow-run` vào cây "Cấu trúc kho mã"**

Khối cây hiện tại (dòng 140-145):

```markdown
├── skills/
│   ├── knowhow-init/      # SKILL.md + references (schema-template, agent-config-snippet)
│   ├── knowhow-capture/   # SKILL.md + references (page-formats)
│   ├── knowhow-distill/   # SKILL.md
│   ├── knowhow-lint/      # SKILL.md + references (schema-review, consolidation-checklist)
│   └── knowhow-query/     # SKILL.md
```

Sửa thành (đổi `knowhow-query` từ nhánh cuối `└──` sang `├──`, thêm `knowhow-run` làm nhánh cuối):

```markdown
├── skills/
│   ├── knowhow-init/      # SKILL.md + references (schema-template, agent-config-snippet)
│   ├── knowhow-capture/   # SKILL.md + references (page-formats)
│   ├── knowhow-distill/   # SKILL.md
│   ├── knowhow-lint/      # SKILL.md + references (schema-review, consolidation-checklist)
│   ├── knowhow-query/     # SKILL.md
│   └── knowhow-run/       # SKILL.md
```

- [ ] **Step 6: Verify**

Run:
```bash
grep -n '5 skills vận hành\|5 skill vận hành\|## 5 skills' README.md
grep -cn 'knowhow-run' README.md
grep -n '6 skills vận hành\|6 skill vận hành\|## 6 skills' README.md
```
Expected:
- Lệnh 1: KHÔNG còn dòng nào nhắc "5 skills/skill vận hành" (output rỗng).
- Lệnh 2: `3` (hàng bảng + cây cấu trúc + có thể chỗ note nếu đếm; tối thiểu 2 hit ở bảng và cây).
- Lệnh 3: có hit cho cả mô tả (dòng 19), heading (dòng 60), note (dòng 95).

- [ ] **Step 7: Commit**

```bash
git add README.md
git commit -m "docs(v1.3): cập nhật README 5 → 6 skills, thêm knowhow-run"
```

---

## Self-Review (đối chiếu plan với spec)

Đối chiếu từng tiêu chí done v1.3 (spec dòng 118-124):

| Tiêu chí done v1.3 | Task phủ |
|---|---|
| `skills/knowhow-run/SKILL.md` tồn tại, rõ ba nhịp + ba dạng input | Task 1 |
| Skill có field `trigger` | Task 2 |
| Skill registry có cột `Khi nào dùng` | Task 3 |
| `rebuild-index` giữ được (đọc `trigger`, sinh cột) | Task 5 (Step 1) |
| `knowhow-distill` điền `trigger` khi tạo/sửa skill | Task 4 |
| `agent-config-snippet.md` có dòng gọi `knowhow-run` | Task 6 |
| README liệt kê 6 skills | Task 7 |
| run load đúng file + làm theo; workflow gọi skill con resolve được | Task 1 (Nhịp 2 + Nhịp 3 mục 2) |
| run KHÔNG ghi gì vào `.knowhow/` | Task 1 (Quy tắc cứng số 1) |

**Đối chiếu "File cần sửa" của spec (dòng 104-114):**

| File spec liệt kê | Xử lý trong plan |
|---|---|
| `skills/knowhow-run/SKILL.md` | Task 1 (tạo mới) |
| `schema-template.md` (trigger + cột Khi nào dùng) | **Chuyển sang `page-formats.md`** (Task 2, 3) — schema-template thực tế không chứa skill frontmatter/registry mẫu (xem mục "Bối cảnh" điểm 5) |
| `page-formats.md` (trigger vào skill page) | Task 2 + Task 3 |
| `knowhow-distill/SKILL.md` | Task 4 |
| `knowhow-lint/SKILL.md` | Task 5 |
| `agent-config-snippet.md` | Task 6 |
| `README.md` | Task 7 |

**Type consistency:** field tên `trigger` (frontmatter), cột tên `Khi nào dùng` (registry) dùng nhất quán xuyên Task 2/3/4/5. Skill vận hành dùng frontmatter `name`+`description` (Task 1); skill sản phẩm dùng `type: skill`+`trigger` (Task 2). Không trộn lẫn.

**Ngoài phạm vi (ghi rõ, không làm):**
- `schema-template.md` dòng 92 còn ghi "4 skill vận hành (init, capture, distill, lint)" — đây là tàn dư cũ trước cả `knowhow-query` (v1.1), KHÔNG thuộc scope v1.3. Để nguyên.
- Project `CLAUDE.md` (file gốc dự án): spec chỉ liệt kê `agent-config-snippet.md` trong bảng "File cần sửa", không liệt kê `CLAUDE.md`. snippet là nguồn được init chèn vào CLAUDE.md, nên sửa snippet là đủ.
