# Knowhow Schema Evolution (v1.2) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Cho hệ Knowhow tự tiến hoá CẤU TRÚC (`SCHEMA.md`) theo từng dự án: distill/query phát hiện "khuôn không vừa", lint tổng hợp và đề xuất sửa khuôn, user duyệt, hệ thống migrate.

**Architecture:** `SCHEMA.md` trở thành "hợp đồng sống" (có `schema_version` + changelog). Ba nguồn tín hiệu strain (distill, query, lint scan) đổ vào một sổ tích luỹ `.knowhow/schema-signals.md`. lint thêm mode `schema-review` đọc sổ + quét sống + áp ngưỡng → đề xuất diff lên SCHEMA → user duyệt → migrate (rewrite link, rebuild index, bump version). Toàn bộ là markdown thuần trong git nên reversible.

**Tech Stack:** Markdown thuần. Các "skill" là file `SKILL.md` chứa chỉ dẫn cho AI agent (Antigravity/Gemini) thực thi. KHÔNG có code biên dịch, KHÔNG có test runner. "Test" = kịch bản mô phỏng trên một `.knowhow/` sandbox trong thư mục tạm, kiểm phần cơ học bằng `grep`/`ls`/`test`, kiểm phần phán đoán bằng checklist đọc.

---

## Bối cảnh cho người thực thi (đọc trước khi bắt đầu)

Bạn là dev giỏi nhưng chưa biết hệ này. Vài điều cốt lõi phải nắm:

1. **Hệ này là gì.** `.knowhow/` là thư mục tri thức của một dự án (markdown thuần). 4 skill vận hành (`knowhow-init`, `knowhow-capture`, `knowhow-distill`, `knowhow-lint`) là các file `SKILL.md` trong `skills/`, chứa chỉ dẫn cho AI agent đọc rồi làm theo. Sửa skill = sửa văn bản chỉ dẫn, không sửa code.

2. **Nguồn chân lý của khuôn là `SCHEMA.md`.** Mọi thay đổi cấu trúc quy về: một diff được duyệt lên `SCHEMA.md` + migrate file bị ảnh hưởng + bump version + ghi changelog.

3. **Bất biến không được phá:** mọi knowhow phải qua `inbox/` trước, KHÔNG ghi thẳng vào `wiki/`. Mọi thay đổi khuôn phải qua user duyệt (không auto-migrate).

4. **Quy ước đường dẫn (chốt từ v1.1):** wiki phẳng, tên file `wiki/<type>-<slug>.md`. Ví dụ `wiki/decision-rest-to-graphql.md`. Link viết `[[rest-to-graphql]]` (slug trần) hoặc `[[decision-rest-to-graphql]]` (kèm type khi trùng slug).

5. **Tách phát hiện khỏi quyết định:** distill và query CHỈ *ghi* tín hiệu vào sổ, KHÔNG tự đổi khuôn. Chỉ lint mode `schema-review` mới tổng hợp + đề xuất. Chỉ user mới duyệt.

6. **Hai kiểu tín hiệu:** tín hiệu *sự kiện* (distill/query, xảy ra liên tục giữa các lần review) được GHI vào sổ. Tín hiệu *trạng thái* (lint: `type-bloat`, `tag-cluster`) là điểm-thời-gian, được tính LIVE trong `schema-review`, KHÔNG ghi trước vào sổ.

### File sẽ tạo / sửa (bản đồ trước khi vào việc)

**Tạo mới:**
- `skills/knowhow-query/SKILL.md` — skill thứ năm: trả lời câu hỏi knowhow + emit `query-miss` + file ngược qua inbox.
- `skills/knowhow-lint/references/schema-review.md` — reference cho mode `schema-review`: ngưỡng kích hoạt 4 loại + migration playbook.

**Sửa:**
- `skills/knowhow-init/references/schema-template.md` — SCHEMA thành hợp đồng sống: `schema_version`, `## Changelog`, mục `schema-signals.md`, ghi chú resolve mở rộng, quy tắc log prefix.
- `skills/knowhow-init/SKILL.md` — sinh SCHEMA có version, tạo `schema-signals.md` rỗng, log prefix.
- `skills/knowhow-capture/SKILL.md` — log prefix (L1).
- `skills/knowhow-distill/SKILL.md` — emit `no-fit-type` + `adhoc-section`, log prefix (L1).
- `skills/knowhow-lint/SKILL.md` — mode `schema-review`, mở rộng resolve `[[slug]]` cho type mới + subfolder, rebuild-index recursive, log prefix (L1).
- `docs/superpowers/specs/task.md` — bổ sung trạng thái v1.2 + 5 kịch bản kiểm chứng.

### Quy ước test chung (dùng lại ở nhiều task)

Nhiều task kết thúc bằng một kịch bản sandbox. Dựng sandbox một lần như sau, và mỗi task chỉ ra phần kiểm riêng:

```bash
# Dựng sandbox sạch trong thư mục tạm
SANDBOX=$(mktemp -d)/proj
mkdir -p "$SANDBOX"
echo "Sandbox: $SANDBOX"
```

Trong sandbox, bạn (đóng vai agent) thực hiện các bước cơ học mà skill mô tả, rồi chạy lệnh kiểm. Phần cần phán đoán của LLM (phân loại item, áp ngưỡng) được kiểm bằng checklist đọc, không bằng bash.

---

## Task 1: SCHEMA.md thành "hợp đồng sống" (schema-template.md)

Biến template SCHEMA thành hợp đồng sống: thêm `schema_version`, `## Changelog`, mô tả `schema-signals.md`, ghi chú resolve mở rộng (chuẩn bị cho type mới/subfolder), và quy tắc log prefix parse được.

**Files:**
- Modify: `skills/knowhow-init/references/schema-template.md`

- [ ] **Step 1: Thêm `schema_version` ngay sau heading title**

Mở `skills/knowhow-init/references/schema-template.md`. Dòng 1 hiện là:

```markdown
# Knowhow Schema — {{PROJECT_NAME}}
```

Thêm ngay sau dòng đó (chèn dòng trống rồi dòng version):

```markdown
# Knowhow Schema — {{PROJECT_NAME}}

**schema_version**: 1
```

- [ ] **Step 2: Thêm `schema-signals.md` vào bảng cấu trúc thư mục**

Trong mục `## Cấu trúc thư mục`, bảng hiện kết thúc ở dòng `| workflows/ | ... |`. Bảng này liệt kê *thư mục*. Thêm một câu ghi chú NGAY DƯỚI bảng (vì `schema-signals.md` là file top-level, không phải thư mục):

```markdown
> **Lưu ý file top-level**: ngoài `SCHEMA.md`, thư mục `.knowhow/` còn có `schema-signals.md` — sổ tích luỹ tín hiệu "khuôn không vừa" (meta về khuôn, KHÔNG phải tri thức). distill và query ghi vào đây; lint `schema-review` đọc. File này KHÔNG nằm trong `wiki/` và KHÔNG vào `index.md`.
```

- [ ] **Step 3: Thêm ghi chú resolve mở rộng vào mục Cross-referencing**

Trong mục `## Cross-referencing`, sau dòng cuối (`- Mọi page phải xuất hiện trong index.md hoặc registry.md tương ứng.`), thêm:

```markdown
- **Type động**: tập type hợp lệ ĐỌC TỪ bảng "Page Types" ở trên, không cố định. Khi schema tiến hoá thêm type mới (ví dụ `experiment`), link `[[experiment-<slug>]]` resolve được ngay.
- **Subfolder**: khi một type bị tách vào subfolder (ví dụ `wiki/experiment/experiment-abc.md`), link vẫn viết `[[experiment-abc]]` (slug trần hoặc kèm type). Resolve tìm đệ quy trong `wiki/`, không chỉ ở mức phẳng.
```

- [ ] **Step 4: Thêm quy tắc log prefix vào Quy tắc vận hành**

Trong mục `## Quy tắc vận hành`, sau dòng `- Mọi hoạt động ghi log vào wiki/log.md`, thêm:

```markdown
- Mỗi entry log dùng heading prefix parse được: `## [YYYY-MM-DD] <op> | <tiêu đề>` (op ∈ init/capture/distill/lint/query). Cho phép `grep "^## \[" wiki/log.md | tail -5` xem hoạt động gần nhất.
```

- [ ] **Step 5: Thêm mục Tiến hoá cấu trúc + Changelog ở cuối file**

Cuối file (sau mục `> **Phạm vi agent**: ...`), thêm hai mục mới:

```markdown

## Tiến hoá cấu trúc (schema evolution)

Khuôn này tự tiến hoá theo dự án. Cơ chế:

1. distill/query phát hiện "khuôn không vừa" → ghi tín hiệu vào `schema-signals.md` (KHÔNG tự đổi khuôn).
2. `knowhow-lint schema-review` đọc sổ + quét sống + áp ngưỡng → đề xuất diff lên SCHEMA.md.
3. User duyệt từng đề xuất.
4. Hệ migrate file bị ảnh hưởng, rewrite link, rebuild index, bump `schema_version`, ghi vào Changelog dưới đây.

Bốn loại thay đổi cấu trúc: thêm/đổi/nghỉ hưu page type, đổi layout (subfolder), đổi format page type, sửa mục SCHEMA. Mọi thay đổi reversible bằng git.

## Glossary & Convention (tiến hoá)

[Thuật ngữ riêng dự án + quy ước vận hành bổ sung. Trống lúc init. schema-review thêm vào khi phát hiện thuật ngữ/quy ước lặp nhiều lần.]

## Changelog

- {{DATE}}: Khởi tạo schema v1 (init).
```

- [ ] **Step 6: Verify nội dung đã thêm đủ**

Run:
```bash
F=skills/knowhow-init/references/schema-template.md
grep -c "schema_version" "$F"          # mong đợi: ≥1
grep -q "schema-signals.md" "$F" && echo "signals OK"
grep -q "Type động" "$F" && echo "resolve-note OK"
grep -q '## \[YYYY-MM-DD\] <op>' "$F" && echo "log-prefix OK"
grep -q "## Changelog" "$F" && echo "changelog OK"
grep -q "## Tiến hoá cấu trúc" "$F" && echo "evolution OK"
grep -q "## Glossary & Convention" "$F" && echo "glossary OK"
```
Expected: in ra `signals OK`, `resolve-note OK`, `log-prefix OK`, `changelog OK`, `evolution OK`, `glossary OK` và số đếm `schema_version` ≥ 1.

- [ ] **Step 7: Commit**

```bash
git add skills/knowhow-init/references/schema-template.md
git commit -m "feat(v1.2): SCHEMA.md thành hợp đồng sống (schema_version + changelog + glossary)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 2: init sinh schema_version + tạo schema-signals.md + log prefix

Cho `knowhow-init` tạo `schema-signals.md` rỗng (đúng format parse được), đảm bảo SCHEMA sinh ra điền `{{DATE}}`, và đổi log entry init sang prefix mới.

**Files:**
- Modify: `skills/knowhow-init/SKILL.md`

- [ ] **Step 1: Thêm schema-signals.md vào cây thư mục mô tả ở Bước 2**

Trong `skills/knowhow-init/SKILL.md`, Bước 2 có khối cây thư mục. Hiện:

```
.knowhow/
├── SCHEMA.md
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

Thay bằng (thêm dòng `schema-signals.md` ngay dưới `SCHEMA.md`):

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

- [ ] **Step 2: Cập nhật Bước 3 để điền `{{DATE}}` vào SCHEMA**

Trong Bước 3 (`Sinh SCHEMA.md`), khối "Thay thế" hiện liệt kê 2 biến. Thêm biến thứ ba. Thay khối:

```markdown
Thay thế:
- `{{PROJECT_NAME}}` → tên dự án user cung cấp
- `{{PROJECT_DESCRIPTION}}` → mô tả domain user cung cấp
```

bằng:

```markdown
Thay thế:
- `{{PROJECT_NAME}}` → tên dự án user cung cấp
- `{{PROJECT_DESCRIPTION}}` → mô tả domain user cung cấp
- `{{DATE}}` → ngày hiện tại (YYYY-MM-DD), dùng cho entry Changelog đầu tiên trong SCHEMA
```

- [ ] **Step 3: Thêm Bước sinh schema-signals.md**

Trong Bước 4 (`Sinh các file nội dung`), sau khối `**workflows/registry.md**`, thêm một khối mới:

````markdown
**schema-signals.md** (top-level, cạnh SCHEMA.md — sổ tích luỹ tín hiệu tiến hoá, tạo rỗng chỉ có header):
```markdown
# Schema Signals

Sổ tích luỹ tín hiệu "khuôn không vừa". Append-only, parse được.
distill và query ghi vào "Đang chờ xử lý". lint `schema-review` đọc, áp ngưỡng, rồi cắt tín hiệu đã dùng sang "Đã xử lý".

Format mỗi dòng:
`- [YYYY-MM-DD] <nguồn> | <loại> | <chi tiết ngắn> | related: <slug-hoặc-tag>`

- nguồn ∈ distill | query
- loại ∈ no-fit-type | adhoc-section | query-miss

## Đang chờ xử lý

## Đã xử lý
```
````

- [ ] **Step 4: Đổi log entry init sang prefix mới (L1)**

Trong Bước 4, khối `**wiki/log.md**` hiện là:

```markdown
# Activity Log

## YYYY-MM-DD
- [init] Khởi tạo .knowhow/ cho dự án {{PROJECT_NAME}}
```

Thay bằng (mỗi op một heading prefix):

```markdown
# Activity Log

## [YYYY-MM-DD] init | Khởi tạo .knowhow/ cho dự án {{PROJECT_NAME}}
```

- [ ] **Step 5: Verify init sinh đúng (sandbox)**

Mô phỏng init: chạy các lệnh cơ học mà skill mô tả vào sandbox, rồi kiểm.

Run:
```bash
SANDBOX=$(mktemp -d)/proj && mkdir -p "$SANDBOX"
cd "$SANDBOX"
mkdir -p .knowhow/{raw,inbox,archive,wiki,skills,workflows}
# tạo schema-signals.md theo template trong Bước 3 của skill
cat > .knowhow/schema-signals.md <<'EOF'
# Schema Signals

## Đang chờ xử lý

## Đã xử lý
EOF
# kiểm cấu trúc
test -f .knowhow/schema-signals.md && echo "signals-file OK"
grep -q "## Đang chờ xử lý" .knowhow/schema-signals.md && echo "pending-section OK"
grep -q "## Đã xử lý" .knowhow/schema-signals.md && echo "processed-section OK"
test ! -d .knowhow/wiki/decisions && echo "wiki-phẳng OK"
cd - >/dev/null
```
Expected: `signals-file OK`, `pending-section OK`, `processed-section OK`, `wiki-phẳng OK`.

- [ ] **Step 6: Verify skill file đã sửa đủ**

Run:
```bash
F=skills/knowhow-init/SKILL.md
grep -q "schema-signals.md" "$F" && echo "signals-step OK"
grep -q '## \[YYYY-MM-DD\] init |' "$F" && echo "log-prefix OK"
grep -q "{{DATE}}" "$F" && echo "date-var OK"
```
Expected: `signals-step OK`, `log-prefix OK`, `date-var OK`.

- [ ] **Step 7: Commit**

```bash
git add skills/knowhow-init/SKILL.md
git commit -m "feat(v1.2): init tạo schema-signals.md + schema_version + log prefix

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 3: Log prefix parse được (L1) cho capture, distill, lint

Đổi format log của các skill còn lại sang prefix `## [YYYY-MM-DD] <op> | <tiêu đề>`. Task 2 đã làm init. Task này làm capture, distill, lint để toàn hệ nhất quán.

**Files:**
- Modify: `skills/knowhow-capture/SKILL.md`
- Modify: `skills/knowhow-distill/SKILL.md`
- Modify: `skills/knowhow-lint/SKILL.md`

- [ ] **Step 1: Sửa capture Bước 5 (Ghi log)**

Trong `skills/knowhow-capture/SKILL.md`, Bước 5 hiện:

```
- YYYY-MM-DD HH:mm [capture] Ghi nhận N items vào inbox từ [nguồn]
```

Thay bằng:

```
## [YYYY-MM-DD] capture | Ghi nhận N items vào inbox từ [nguồn]
```

- [ ] **Step 2: Sửa distill Bước 6 (Log)**

Trong `skills/knowhow-distill/SKILL.md`, Bước 5 mục 6 (`**Log**`), khối hiện:

```
- [distill] Tạo mới [type]/[slug].md
- [distill] Cập nhật [type]/[slug].md: [mô tả thay đổi]
- [distill] Gộp [slug-a].md + [slug-b].md → [slug-mới].md
- [distill] Bỏ qua inbox/[slug].md: [lý do]
```

Thay bằng:

```
## [YYYY-MM-DD] distill | Tạo mới wiki/<type>-<slug>.md
## [YYYY-MM-DD] distill | Cập nhật wiki/<type>-<slug>.md: <mô tả thay đổi>
## [YYYY-MM-DD] distill | Gộp <slug-a> + <slug-b> → <slug-mới>
## [YYYY-MM-DD] distill | Bỏ qua inbox/<slug>.md: <lý do>
```

- [ ] **Step 3: Sửa lint mục Log**

Trong `skills/knowhow-lint/SKILL.md`, mục `## Log` ở cuối file, khối hiện:

```
- [lint] Quick lint: N vấn đề phát hiện, M đã fix
- [lint] Consolidation: N vấn đề phát hiện, M đã fix
```

Thay bằng:

```
## [YYYY-MM-DD] lint | Quick lint: N vấn đề, M đã fix
## [YYYY-MM-DD] lint | Consolidation: N vấn đề, M đã fix
```

Đồng thời, trong `## Chế độ 3: Rebuild Index`, Bước 5 hiện:

```
5. Ghi log: `- [lint] Rebuild index + 2 registry từ frontmatter`.
```

Thay bằng:

```
5. Ghi log: `## [YYYY-MM-DD] lint | Rebuild index + 2 registry từ frontmatter`.
```

- [ ] **Step 4: Verify cả 3 file dùng prefix mới, không còn format cũ**

Run:
```bash
for F in skills/knowhow-capture/SKILL.md skills/knowhow-distill/SKILL.md skills/knowhow-lint/SKILL.md; do
  echo "== $F =="
  grep -c '## \[YYYY-MM-DD\]' "$F"      # mong đợi: ≥1
done
# không còn format log cũ kiểu "- [distill]" / "- [lint]" / "- [capture]" trong dòng log
! grep -qE '^\s*- \[(capture|distill|lint)\] ' skills/knowhow-capture/SKILL.md skills/knowhow-distill/SKILL.md skills/knowhow-lint/SKILL.md && echo "no-old-log-format OK"
```
Expected: mỗi file đếm `## [YYYY-MM-DD]` ≥ 1, và `no-old-log-format OK`.

- [ ] **Step 5: Verify grep log thực tế hoạt động (sandbox)**

Run:
```bash
SANDBOX=$(mktemp -d) && cd "$SANDBOX"
cat > log.md <<'EOF'
# Activity Log

## [2026-06-05] init | Khởi tạo
## [2026-06-06] capture | Ghi 3 items
## [2026-06-07] distill | Tạo mới wiki/pattern-retry.md
EOF
grep "^## \[" log.md | tail -2
cd - >/dev/null
```
Expected: in ra đúng 2 dòng cuối (`capture` và `distill`), chứng tỏ log parse + grep được.

- [ ] **Step 6: Commit**

```bash
git add skills/knowhow-capture/SKILL.md skills/knowhow-distill/SKILL.md skills/knowhow-lint/SKILL.md
git commit -m "feat(v1.2,L1): log prefix parse được cho capture/distill/lint

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 4: distill emit tín hiệu strain (no-fit-type + adhoc-section)

Cho `knowhow-distill` ghi tín hiệu vào `schema-signals.md` khi phát hiện khuôn không vừa: item không thuộc 6 type (`no-fit-type`), hoặc phải thêm section ngoài template chuẩn (`adhoc-section`). distill CHỈ ghi, KHÔNG tự đổi khuôn.

**Files:**
- Modify: `skills/knowhow-distill/SKILL.md`

- [ ] **Step 1: Thêm Bước 3.5 "Phát tín hiệu strain" vào Flow distill**

Trong `skills/knowhow-distill/SKILL.md`, mục `### Bước 3: Phân tích và đề xuất` kết thúc trước `### Bước 4: User duyệt`. Chèn một mục mới NGAY TRƯỚC `### Bước 4`:

````markdown
### Bước 3.5: Phát tín hiệu strain (tiến hoá cấu trúc)

Trong lúc phân loại + đề xuất, để ý hai dấu hiệu "khuôn không vừa". Nếu gặp, GHI tín hiệu vào `.knowhow/schema-signals.md` (mục "Đang chờ xử lý"). KHÔNG tự đổi khuôn — đó là việc của `knowhow-lint schema-review`.

**Khi nào emit `no-fit-type`**: item rõ ràng là một *loại tri thức* không nằm trong 4 wiki type {decision, pattern, concept, troubleshooting} nhưng buộc phải xếp tạm thành wiki page chung chung. Ví dụ: "kết quả thí nghiệm", "nguồn tham khảo cần lưu", "runbook vận hành". Dấu hiệu: bạn thấy mình miễn cưỡng chọn type vì không cái nào khớp.

**Khi nào emit `adhoc-section`**: khi tạo/cập nhật page, bạn phải thêm một section KHÔNG có trong template chuẩn của type đó (xem `../knowhow-capture/references/page-formats.md`), và bạn nhận ra section này từng xuất hiện ở page khác cùng type.

**Cách ghi** (append vào mục "Đang chờ xử lý" của `schema-signals.md`):
```bash
echo '- [YYYY-MM-DD] distill | no-fit-type | <chi tiết ngắn> | related: tag:<chủ-đề>' >> .knowhow/schema-signals.md
echo '- [YYYY-MM-DD] distill | adhoc-section | section "<tên section>" ở page <slug> | related: tag:<chủ-đề>' >> .knowhow/schema-signals.md
```

Ví dụ:
```
- [2026-06-05] distill | no-fit-type | item về kết quả thí nghiệm, không vừa decision/pattern/concept/troubleshooting | related: tag:experiment
- [2026-06-05] distill | adhoc-section | section "Metrics đo được" ở page experiment-ab-test | related: tag:experiment
```

> **Quan trọng**: emit tín hiệu KHÔNG chặn distill. Vẫn xếp item vào wiki page tốt nhất hiện có và xử lý bình thường. Tín hiệu chỉ là ghi chú cho lần `schema-review` sau.
````

- [ ] **Step 2: Verify nội dung đã thêm**

Run:
```bash
F=skills/knowhow-distill/SKILL.md
grep -q "Bước 3.5: Phát tín hiệu strain" "$F" && echo "step35 OK"
grep -q "no-fit-type" "$F" && echo "no-fit-type OK"
grep -q "adhoc-section" "$F" && echo "adhoc-section OK"
grep -q "schema-signals.md" "$F" && echo "writes-to-signals OK"
grep -q "KHÔNG tự đổi khuôn" "$F" && echo "separation OK"
```
Expected: `step35 OK`, `no-fit-type OK`, `adhoc-section OK`, `writes-to-signals OK`, `separation OK`.

- [ ] **Step 3: Verify emit ghi đúng format (sandbox)**

Run:
```bash
SANDBOX=$(mktemp -d) && cd "$SANDBOX"
cat > schema-signals.md <<'EOF'
# Schema Signals

## Đang chờ xử lý

## Đã xử lý
EOF
# mô phỏng distill emit
echo '- [2026-06-05] distill | no-fit-type | item kết quả thí nghiệm | related: tag:experiment' >> schema-signals.md
# kiểm format parse được: 5 trường ngăn bởi " | "
LINE=$(grep "no-fit-type" schema-signals.md)
echo "$LINE" | grep -qE '^- \[[0-9]{4}-[0-9]{2}-[0-9]{2}\] distill \| no-fit-type \| .+ \| related: ' && echo "format OK"
cd - >/dev/null
```
Expected: `format OK`.

- [ ] **Step 4: Commit**

```bash
git add skills/knowhow-distill/SKILL.md
git commit -m "feat(v1.2): distill emit tín hiệu no-fit-type + adhoc-section vào sổ

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 5: Skill mới `knowhow-query`

Tạo skill thứ năm: trả lời câu hỏi nhắm vào knowhow, kèm trích dẫn; emit `query-miss` khi phải chắp vá ≥ 3 page hoặc không có page sạch; nếu câu trả lời đáng tái dùng, file ngược qua `inbox/` (KHÔNG ghi thẳng wiki) và lưu Q&A vào `raw/`.

**Files:**
- Create: `skills/knowhow-query/SKILL.md`

- [ ] **Step 1: Tạo file skill knowhow-query**

Tạo `skills/knowhow-query/SKILL.md` với nội dung đầy đủ:

````markdown
---
name: knowhow-query
description: "Trả lời câu hỏi nhắm vào knowhow dự án: đọc index + grep page liên quan, tổng hợp câu trả lời kèm trích dẫn [[slug]]. Phát tín hiệu query-miss khi hỏi không trúng page sạch. Nếu câu trả lời đáng tái dùng, file ngược qua inbox (không ghi thẳng wiki). Trigger: 'tra knowhow', 'dự án có gì về X', 'so sánh...', 'knowhow query', khi user hỏi một câu nhắm vào tri thức đã tích luỹ."
---

# Knowhow Query

Trả lời câu hỏi từ knowhow đã tích luỹ. Đồng thời là nguồn tín hiệu thứ ba cho tiến hoá cấu trúc: câu hỏi lặp mà không trúng page là dấu hiệu mạnh của thiếu cấu trúc.

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user chạy `knowhow-init` trước.

## Phạm vi agent

Như 4 skill kia, `knowhow-query` viết cho Antigravity (Gemini). Agent khác đọc được kết quả (markdown thuần) nhưng chưa chạy được skill cho tới khi port.

## Flow query

### Bước 1: Tìm page liên quan

1. Đọc `wiki/index.md` + `skills/registry.md` + `workflows/registry.md` để biết tổng quan.
2. Rút 3-5 từ khoá từ câu hỏi. Grep tìm page liên quan:
   ```bash
   grep -ril "<từ khoá>" .knowhow/wiki .knowhow/skills .knowhow/workflows
   ```
3. Đọc các file hit.

### Bước 2: Tổng hợp câu trả lời

- Tổng hợp từ các page đã đọc, trả lời trực tiếp câu hỏi.
- Trích dẫn nguồn bằng `[[slug]]` cho mỗi tuyên bố lấy từ một page cụ thể.
- Nếu không tìm thấy page nào liên quan, nói thẳng "chưa có knowhow về việc này" thay vì bịa.

### Bước 3: Phát tín hiệu query-miss

Đánh giá chất lượng câu trả lời. Emit `query-miss` vào `.knowhow/schema-signals.md` (mục "Đang chờ xử lý") khi MỘT trong hai:

- Phải chắp vá từ **≥ 3 page** mới trả lời được (T = 3, khởi điểm).
- KHÔNG có page sạch nào trực tiếp trả lời (toàn suy luận chắp nối).

Cách ghi:
```bash
echo '- [YYYY-MM-DD] query | query-miss | <câu hỏi rút gọn>, phải chắp <N> page | related: tag:<chủ-đề>' >> .knowhow/schema-signals.md
```

Ví dụ:
```
- [2026-06-07] query | query-miss | "so sánh 3 thí nghiệm A/B/C", phải chắp 4 page | related: tag:experiment
```

> Nếu câu trả lời gọn (1-2 page sạch trúng đích), KHÔNG emit. Tín hiệu chỉ dành cho "khuôn không vừa".

### Bước 4: File ngược qua inbox (tôn trọng cửa duy nhất)

Nếu câu trả lời đáng tái dùng (user xác nhận OK), thả nó vào `inbox/` như một candidate page. KHÔNG ghi thẳng vào `wiki/`. distill xử lý sau như mọi item khác.

1. Hỏi user: "Câu trả lời này đáng lưu lại không?"
2. Nếu user OK:
   - Lưu Q&A nguyên văn vào `raw/YYYY-MM-DD-query-<slug>.md` (provenance, nhất quán quy tắc capture C3).
   - Tạo inbox item `inbox/YYYY-MM-DD-query-<slug>.md` theo format inbox trong `../knowhow-capture/references/page-formats.md`, với:
     - `captured_from: query`
     - `source_file: raw/YYYY-MM-DD-query-<slug>.md`
   - KHÔNG ghi vào `wiki/`, `skills/`, `workflows/`.

### Bước 5: Ghi log

Thêm vào `wiki/log.md`:
```
## [YYYY-MM-DD] query | <câu hỏi rút gọn> (emit query-miss: có/không, file inbox: có/không)
```

## Quy tắc cứng

1. query KHÔNG ghi thẳng vào `wiki/`. Mọi knowhow mới đi qua `inbox/` → distill (giữ bất biến "cửa duy nhất").
2. Không bịa câu trả lời. Không có page thì nói không có.
3. Trích dẫn `[[slug]]` cho mọi tuyên bố lấy từ page.
4. Chỉ emit `query-miss` khi thật sự "khuôn không vừa" (≥3 page hoặc không page sạch), tránh báo nhiễu.
````

- [ ] **Step 2: Verify skill có cấu trúc đúng**

Run:
```bash
F=skills/knowhow-query/SKILL.md
test -f "$F" && echo "file OK"
head -3 "$F" | grep -q "name: knowhow-query" && echo "name OK"
grep -q "query-miss" "$F" && echo "query-miss OK"
grep -q "File ngược qua inbox" "$F" && echo "inbox OK"
grep -q "KHÔNG ghi thẳng" "$F" && echo "invariant OK"
grep -q "raw/YYYY-MM-DD-query" "$F" && echo "provenance OK"
grep -q '## \[YYYY-MM-DD\] query |' "$F" && echo "log-prefix OK"
```
Expected: `file OK`, `name OK`, `query-miss OK`, `inbox OK`, `invariant OK`, `provenance OK`, `log-prefix OK`.

- [ ] **Step 3: Commit**

```bash
git add skills/knowhow-query/SKILL.md
git commit -m "feat(v1.2): skill knowhow-query (trả lời + emit query-miss + file ngược inbox)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 6: Reference cho lint `schema-review` (ngưỡng + migration playbook)

Tạo reference file chứa 4 ngưỡng kích hoạt và migration playbook 4 loại. Tách ra reference để `SKILL.md` của lint gọn (theo nguyên tắc file focused). Task 7 sẽ móc reference này vào lint SKILL.

**Files:**
- Create: `skills/knowhow-lint/references/schema-review.md`

- [ ] **Step 1: Tạo reference schema-review.md**

Tạo `skills/knowhow-lint/references/schema-review.md`:

````markdown
# Schema Review: Ngưỡng & Migration Playbook

Reference cho `knowhow-lint` mode `schema-review`. Chứa: ngưỡng kích hoạt 4 loại thay đổi cấu trúc, và migration playbook tương ứng.

Nguyên tắc nền: **tách phát hiện khỏi quyết định**. lint tổng hợp + đề xuất; chỉ user duyệt; mọi migrate reversible bằng git.

---

## 1. Hai kiểu tín hiệu

- **Tín hiệu sự kiện** (đọc từ `schema-signals.md` mục "Đang chờ xử lý"): `no-fit-type`, `adhoc-section` (distill), `query-miss` (query). Đã được ghi sẵn giữa các lần review.
- **Tín hiệu trạng thái** (tính LIVE lúc review, KHÔNG đọc từ sổ): `type-bloat`, `tag-cluster`. Quét tại thời điểm chạy.

## 2. Quét sống (tính tín hiệu trạng thái)

```bash
# Đếm page mỗi type (dựa prefix tên file, đệ quy cả subfolder)
for t in decision pattern concept troubleshooting; do
  n=$(find .knowhow/wiki -name "${t}-*.md" | wc -l | tr -d ' ')
  echo "$t: $n"
done
# Cụm tag: gom tag từ frontmatter mọi wiki page, đếm tần suất
grep -rh "^tags:" .knowhow/wiki | sed 's/tags://' | tr -d '[]' | tr ',' '\n' | sed 's/^ *//' | sort | uniq -c | sort -rn
# Orphan ratio: page không được [[link]] hay related trỏ tới (tham khảo consolidation checklist mục 8)
```

- `type-bloat`: một type vượt 30 page (xem ngưỡng đổi layout).
- `tag-cluster`: một tag xuất hiện ở ≥ 5 page mà chưa thành type riêng.

## 3. Ngưỡng kích hoạt (giá trị khởi điểm, bảo thủ để tránh nhiễu)

| Loại thay đổi | Ngưỡng kích hoạt |
|---|---|
| **Thêm page type** | ≥ 5 tín hiệu `no-fit-type`/`tag-cluster` cùng chủ đề, HOẶC ≥ 5 page default-wiki chung một cụm tag → đề xuất "phong" cụm thành type mới |
| **Đổi layout (subfolder)** | 1 type vượt 30 page phẳng → đề xuất gắt vào subfolder nhóm theo tag |
| **Đổi format page type** | cùng một section tự chế (`adhoc-section`) xuất hiện ở ≥ 4 page cùng type → đề xuất thêm vào template type đó |
| **Thêm mục SCHEMA.md** | thuật ngữ/quy ước lặp lại nhiều lần trong body các page → đề xuất thêm vào mục Glossary & Convention |

Lưới an toàn: tín hiệu CHƯA đủ ngưỡng vẫn tích luỹ trong sổ, không mất, không đề xuất.

## 4. Migration playbook (4 loại)

Mỗi đề xuất được duyệt chạy một batch migrate. Mọi batch ĐỀU kết thúc bằng phần "Bước chung cuối batch" ở dưới.

### 4.1. Thêm type

1. Định nghĩa type mới trong `SCHEMA.md`: thêm dòng vào bảng "Page Types" (`<type> | wiki/<type>-<slug>.md | <mục đích>`) và Naming Conventions.
2. Reclassify page default cũ: đề xuất **TỪNG FILE MỘT**, user duyệt từng file. KHÔNG đổi hàng loạt. Mỗi file được duyệt: đổi tên `wiki/<old>-<slug>.md` → `wiki/<type>-<slug>.md`, cập nhật `type:` trong frontmatter.
3. (Tín hiệu liên quan: `no-fit-type`, `tag-cluster`, `query-miss` cùng chủ đề.)

### 4.2. Đổi layout (subfolder)

1. Tạo subfolder `wiki/<type>/` (hoặc nhóm theo tag).
2. Move các file của type đó vào subfolder.
3. **Rewrite mọi `[[link]]`** trỏ tới (xem grep ở Bước chung).
4. Cập nhật ghi chú resolve trong `SCHEMA.md` (mục Cross-referencing đã nói resolve tìm đệ quy — xác nhận còn đúng).

### 4.3. Đổi format page type

1. Thêm section vào template type trong `../knowhow-capture/references/page-formats.md` (áp cho page tạo MỚI).
2. Backfill page cũ là **TUỲ CHỌN**: đề xuất riêng từng đợt, KHÔNG tự động hàng loạt.
3. (Tín hiệu liên quan: `adhoc-section`.)

### 4.4. Thêm mục SCHEMA

1. Sửa text `SCHEMA.md`: thêm thuật ngữ vào mục "Glossary & Convention" hoặc quy ước vào "Quy tắc vận hành".

### Type nghỉ hưu (retire)

Page của type bị nghỉ hưu: set `status: archived` (tái dùng field v1.1 C4), move vào `archive/`. Xoá định nghĩa type khỏi bảng Page Types.

## 5. Bước chung cuối mỗi batch migrate

Bắt buộc, theo đúng thứ tự:

1. **Bump `schema_version`** ở đầu `SCHEMA.md` (tăng 1).
2. **Ghi Changelog SCHEMA**: thêm dòng `- YYYY-MM-DD: <loại thay đổi> — <lý do ngắn>` vào mục `## Changelog`.
3. **Rewrite link ảnh hưởng**: grep + sửa mọi `[[old-slug]]`:
   ```bash
   grep -rln "\[\[<old-slug>\]\]" .knowhow
   ```
4. **Rebuild index**: chạy mode `rebuild-index` (sinh lại `wiki/index.md` + 2 registry từ frontmatter).
5. **Ghi log**: `## [YYYY-MM-DD] lint | Schema-review: <loại>, <N> file migrate`.
6. **Đánh dấu tín hiệu đã xử lý**: cắt các dòng tín hiệu liên quan từ "Đang chờ xử lý" sang "Đã xử lý" trong `schema-signals.md` (để không đếm lại lần sau).

> **Reversibility**: toàn bộ là markdown trong git. Nếu migrate sai, `git revert` đưa về trạng thái cũ sạch.

## 6. Phụ thuộc v1.1 (không bỏ sót)

- **Tái dùng** `rebuild-index` (v1.1 P2-F) làm bước cuối mỗi migrate.
- **Phải mở rộng** resolve `[[slug]]` (v1.1 P0-B) cho type mới + subfolder TRƯỚC khi migrate sinh chúng — nếu không, lint báo link hỏng giả. (Xem `knowhow-lint/SKILL.md` mục 1b.)
- Type nghỉ hưu dùng `status: archived` + `archive/` (v1.1 C4).
````

- [ ] **Step 2: Verify reference đầy đủ**

Run:
```bash
F=skills/knowhow-lint/references/schema-review.md
test -f "$F" && echo "file OK"
grep -q "Ngưỡng kích hoạt" "$F" && echo "thresholds OK"
grep -q "Migration playbook" "$F" && echo "migration OK"
grep -q "Bump .schema_version" "$F" && echo "bump OK"
grep -q "Đánh dấu tín hiệu đã xử lý" "$F" && echo "mark-processed OK"
grep -q "type-bloat" "$F" && echo "live-signal OK"
grep -q "git revert" "$F" && echo "reversible OK"
# 4 loại thay đổi đều có
for k in "Thêm type" "Đổi layout" "Đổi format" "Thêm mục SCHEMA"; do
  grep -q "$k" "$F" && echo "loại '$k' OK"
done
```
Expected: tất cả dòng `... OK`.

- [ ] **Step 3: Commit**

```bash
git add skills/knowhow-lint/references/schema-review.md
git commit -m "feat(v1.2): reference schema-review (ngưỡng + migration playbook 4 loại)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 7: Mở rộng resolve `[[slug]]` cho type mới + subfolder

`schema-review` migrate có thể sinh type mới và subfolder. Resolve hiện cứng 4 type + wiki phẳng → sẽ báo link hỏng giả. Mở rộng resolve TRƯỚC khi móc migration vào (Task 8). Cũng cập nhật `rebuild-index` quét đệ quy.

**Files:**
- Modify: `skills/knowhow-lint/SKILL.md`

- [ ] **Step 1: Mở rộng thuật toán resolve ở mục 1b**

Trong `skills/knowhow-lint/SKILL.md`, mục `### 1b. Link integrity`, khối thuật toán resolve hiện liệt kê 3 bước (1: `<type>-<slug>`, 2: slug trần, 3: skill/workflow) với type CỨNG `{decision, pattern, concept, troubleshooting}` và đường dẫn phẳng `wiki/<type>-<slug>.md`.

Thay toàn bộ khối từ `- Resolve mỗi link `[[X]]` theo thuật toán ...` đến hết bước 3 bằng:

````markdown
- Trước khi resolve, **đọc tập type hợp lệ từ `SCHEMA.md`** (cột "Type" trong bảng Page Types) thay vì hardcode. Gọi tập này là `TYPES` (mặc định mới init: {decision, pattern, concept, troubleshooting}; sau tiến hoá có thể thêm, ví dụ `experiment`).
- File wiki có thể nằm **phẳng** (`wiki/<type>-<slug>.md`) hoặc **trong subfolder** (`wiki/<nhóm>/<type>-<slug>.md`). Resolve tìm ĐỆ QUY trong `wiki/`.
- Resolve mỗi link `[[X]]` theo thuật toán (match CHẶT type-slug, không glob đuôi tự do):
  1. Nếu `X` có dạng `<type>-<slug>` với `type ∈ TYPES`: tìm đệ quy file khớp chính xác tên `<type>-<slug>.md` trong `wiki/`:
     ```bash
     find .knowhow/wiki -name "<type>-<slug>.md"
     ```
     Đúng 1 file → resolve OK. 0 → báo link hỏng. ≥2 (cùng tên ở 2 subfolder) → báo trùng vị trí.
  2. Nếu `X` là slug trần: với mỗi `type ∈ TYPES`, tìm đệ quy `find .knowhow/wiki -name "<type>-X.md"`. Đếm tổng số file tồn tại:
     - Đúng 1 → resolve OK.
     - 0 → báo link hỏng (ghi page nguồn + target).
     - ≥2 (cùng slug khác type) → báo **ambiguous**, yêu cầu link kèm type `[[<type>-X]]`.
  3. Skill/workflow: tìm đệ quy `find .knowhow/skills -name "X.md"` hoặc `find .knowhow/workflows -name "X.md"`.
````

- [ ] **Step 2: Cập nhật rebuild-index quét đệ quy + type động**

Trong mục `## Chế độ 3: Rebuild Index`, Bước 1 và 2 hiện giả định wiki phẳng + 4 heading cứng. Thay Bước 1-2:

```markdown
1. Quét frontmatter mọi file trong `wiki/` ĐỆ QUY (gồm subfolder, trừ index.md, log.md): đọc `type`, `title`, `tags`, `updated`. Dùng `find .knowhow/wiki -name "*.md"`.
2. Sinh `wiki/index.md`, group theo `type`. Tập type ĐỌC TỪ bảng Page Types trong `SCHEMA.md` (không cố định 4 heading — nếu schema đã thêm type mới như `experiment`, sinh thêm heading tương ứng). Mỗi dòng `- [[<type>-<slug>]] - <title>`.
```

Tương tự, Bước 3-4 (skills/workflows) đổi quét đệ quy: thay `Quét frontmatter mọi file trong `skills/`` thành `Quét frontmatter mọi file trong `skills/` (đệ quy, `find .knowhow/skills -name "*.md"`)`, và tương tự cho `workflows/`.

- [ ] **Step 3: Verify nội dung resolve mở rộng**

Run:
```bash
F=skills/knowhow-lint/SKILL.md
grep -q "đọc tập type hợp lệ từ" "$F" && echo "dynamic-types OK"
grep -q "tìm ĐỆ QUY trong" "$F" && echo "recursive OK"
grep -q 'find .knowhow/wiki -name' "$F" && echo "find-cmd OK"
grep -q "TYPES" "$F" && echo "types-set OK"
```
Expected: `dynamic-types OK`, `recursive OK`, `find-cmd OK`, `types-set OK`.

- [ ] **Step 4: Verify resolve đệ quy + type mới hoạt động (sandbox)**

Mô phỏng một wiki có subfolder và type mới `experiment`, kiểm resolve không báo hỏng giả.

Run:
```bash
SANDBOX=$(mktemp -d) && cd "$SANDBOX"
mkdir -p .knowhow/wiki/experiment
# page type mới trong subfolder
cat > .knowhow/wiki/experiment/experiment-ab-test.md <<'EOF'
---
type: experiment
title: "A/B test nút mua"
---
nội dung
EOF
# page phẳng trỏ link slug trần tới page trong subfolder
cat > .knowhow/wiki/decision-use-ab.md <<'EOF'
---
type: decision
title: "Dùng A/B testing"
---
Xem thêm [[ab-test]] và [[experiment-ab-test]].
EOF
# resolve slug trần "ab-test" (type experiment) — tìm đệ quy
find .knowhow/wiki -name "experiment-ab-test.md" | grep -q . && echo "resolve-recursive OK"
# resolve dạng tường minh experiment-ab-test
test -f "$(find .knowhow/wiki -name 'experiment-ab-test.md')" && echo "resolve-explicit OK"
cd - >/dev/null
```
Expected: `resolve-recursive OK`, `resolve-explicit OK`. (Đây kiểm cơ chế `find` đệ quy mà thuật toán dựa vào; phần đọc TYPES từ SCHEMA là phán đoán của agent, kiểm bằng đọc Step 3.)

- [ ] **Step 5: Commit**

```bash
git add skills/knowhow-lint/SKILL.md
git commit -m "feat(v1.2,P0-B): resolve [[slug]] hỗ trợ type động + subfolder đệ quy

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 8: lint mode `schema-review` (móc detection + migration vào SKILL)

Thêm chế độ thứ tư vào `knowhow-lint`: đọc sổ tín hiệu + quét sống + áp ngưỡng (Task 6 reference) + đề xuất diff → user duyệt → migrate (Task 6 playbook). Cập nhật phần "Chọn chế độ" để route trigger.

**Files:**
- Modify: `skills/knowhow-lint/SKILL.md`

- [ ] **Step 1: Thêm route cho schema-review vào "Chọn chế độ"**

Trong `skills/knowhow-lint/SKILL.md`, mục `## Chọn chế độ`, sau dòng `- User nói "rebuild", "rebuild-index", "sinh lại index" → chạy **Rebuild Index**.`, thêm:

```markdown
- User nói "schema-review", "review khuôn", "tiến hoá khuôn", "schema evolution" → chạy **Schema Review**.
```

- [ ] **Step 2: Thêm mục mode Schema Review**

Sau toàn bộ `## Chế độ 3: Rebuild Index` (trước mục `## Log` ở cuối file), chèn:

````markdown
---

## Chế độ 4: Schema Review (tiến hoá cấu trúc)

Tổng hợp tín hiệu "khuôn không vừa" → đề xuất diff lên `SCHEMA.md` → user duyệt → migrate. Đây là vòng lặp tiến hoá *bên trên* cơ chế cải tiến nội dung.

**Reference bắt buộc đọc trước**: `references/schema-review.md` (ngưỡng + migration playbook + bước chung cuối batch).

### Flow

1. **Đọc sổ tín hiệu**: đọc `.knowhow/schema-signals.md`, lấy các dòng trong mục "Đang chờ xử lý" (BỎ QUA mục "Đã xử lý").
2. **Quét sống**: tính tín hiệu trạng thái tại thời điểm chạy (đếm page mỗi type, cụm tag, file/folder phình, orphan). Lệnh cụ thể trong `references/schema-review.md` mục 2. KHÔNG ghi tín hiệu trạng thái vào sổ.
3. **Áp ngưỡng**: đối chiếu tín hiệu (sự kiện + trạng thái) với 4 ngưỡng trong reference mục 3. Chỉ những gì VƯỢT ngưỡng mới thành đề xuất. Dưới ngưỡng → giữ trong sổ, không báo.
4. **Sinh đề xuất diff**, nhóm theo 4 loại thay đổi. Trình bày cho user (xem Output format dưới).
5. **User duyệt từng đề xuất**: đồng ý / sửa / bác. Bác → tín hiệu ở lại "Đang chờ xử lý".
6. **Migrate** cái được duyệt theo playbook reference mục 4, kết thúc bằng "Bước chung cuối batch" (reference mục 5: bump version, ghi changelog, rewrite link, rebuild index, ghi log, cắt tín hiệu sang "Đã xử lý").

### Output format (Schema Review)

```markdown
## Schema Review Report - YYYY-MM-DD

### Thêm page type (N đề xuất)
- [ ] Cụm tag `experiment` có 6 tín hiệu no-fit-type + 5 page → phong thành type `experiment`?
      Diff SCHEMA: thêm dòng `experiment | wiki/experiment-<slug>.md | Kết quả thí nghiệm`.
      Migrate: reclassify 5 page (duyệt từng file).

### Đổi layout (N đề xuất)
- [ ] Type `pattern` có 34 page phẳng (>30) → gắt vào `wiki/pattern/`?

### Đổi format (N đề xuất)
- [ ] Section "Metrics" xuất hiện ở 4 page type experiment → thêm vào template?

### Thêm mục SCHEMA (N đề xuất)
- [ ] Thuật ngữ "spike" lặp 7 lần → thêm vào Glossary?

✅ Không đủ ngưỡng nào (nếu chưa có gì vượt)
```

### Nguyên tắc

- **Tách phát hiện khỏi quyết định**: lint chỉ đề xuất; chỉ user duyệt.
- **Ngưỡng bảo thủ**: thà bỏ sót còn hơn báo nhiễu lúc đầu (tín hiệu không mất, vẫn tích luỹ).
- **Reversibility**: mọi migrate là markdown trong git, `git revert` được.
- **Không auto-migrate**: mọi batch gate qua user.
````

- [ ] **Step 3: Verify mode đã móc đủ**

Run:
```bash
F=skills/knowhow-lint/SKILL.md
grep -q "Schema Review" "$F" && echo "mode OK"
grep -q "schema-review" "$F" && echo "route OK"
grep -q "references/schema-review.md" "$F" && echo "ref-link OK"
grep -q "Đang chờ xử lý" "$F" && echo "reads-pending OK"
grep -q "Output format (Schema Review)" "$F" && echo "output OK"
grep -q "Không auto-migrate" "$F" && echo "gated OK"
```
Expected: tất cả `... OK`.

- [ ] **Step 4: Verify đếm page theo type + cụm tag (sandbox)**

Kiểm các lệnh quét sống mà reference dựa vào thực sự chạy.

Run:
```bash
SANDBOX=$(mktemp -d) && cd "$SANDBOX"
mkdir -p .knowhow/wiki
for i in 1 2 3 4 5; do
cat > .knowhow/wiki/concept-exp-$i.md <<EOF
---
type: concept
title: "exp $i"
tags: [experiment, ml]
---
EOF
done
# đếm page type concept
n=$(find .knowhow/wiki -name "concept-*.md" | wc -l | tr -d ' ')
[ "$n" -eq 5 ] && echo "count-type OK ($n)"
# cụm tag: experiment phải xuất hiện 5 lần
c=$(grep -rh "^tags:" .knowhow/wiki | tr -d '[]' | tr ',' '\n' | grep -c "experiment")
[ "$c" -eq 5 ] && echo "tag-cluster OK ($c)"
cd - >/dev/null
```
Expected: `count-type OK (5)`, `tag-cluster OK (5)`.

- [ ] **Step 5: Commit**

```bash
git add skills/knowhow-lint/SKILL.md
git commit -m "feat(v1.2): lint mode schema-review (đọc sổ + quét sống + ngưỡng + migrate)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 9: Cập nhật task.md (trạng thái v1.2 + 5 kịch bản kiểm chứng)

Bổ sung `docs/superpowers/specs/task.md`: trạng thái skill mới + 5 kịch bản kiểm chứng v1.2 từ spec, để Task 10 chạy theo.

**Files:**
- Modify: `docs/superpowers/specs/task.md`

- [ ] **Step 1: Thêm dòng knowhow-query vào bảng trạng thái**

Trong `docs/superpowers/specs/task.md`, bảng `## Trạng thái implement`, sau dòng `| knowhow-lint | Done (v1.1) | ... |`, thêm:

```markdown
| knowhow-query | Done (v1.2) | skills/knowhow-query/SKILL.md |
```

- [ ] **Step 2: Thêm mục kịch bản kiểm chứng v1.2**

Cuối file `task.md`, sau mục `## Tiêu chí pass`, thêm:

````markdown

---

## Test end-to-end v1.2 (schema evolution)

Chạy trên 1 dự án nghiên cứu giả lập (thư mục tạm). Mỗi kịch bản kiểm một loại thay đổi.

1. **Living contract + signals (nền)**: chạy `knowhow-init`. Kiểm:
   - `.knowhow/SCHEMA.md` có `**schema_version**: 1` và mục `## Changelog`.
   - `.knowhow/schema-signals.md` tồn tại, có `## Đang chờ xử lý` + `## Đã xử lý`, rỗng.
   - SCHEMA có mục `## Glossary & Convention` và `## Tiến hoá cấu trúc`.

2. **Thêm type (luồng chính)**: capture nhiều item kiểu "experiment" → distill emit `no-fit-type` đúng format vào sổ. Sau ≥ 5 tín hiệu cùng chủ đề, chạy `knowhow-lint schema-review` → kiểm đề xuất type `experiment` xuất hiện. Duyệt → kiểm: SCHEMA bảng Page Types có dòng `experiment`, `schema_version` bump lên 2, Changelog SCHEMA có entry, naming `wiki/experiment-<slug>.md` đúng, index rebuild đúng, resolve `[[slug]]` vẫn chạy, không link hỏng.

3. **Query làm tín hiệu**: lặp một câu query không trúng page sạch (≥ 3 page chắp vá) → `knowhow-query` emit `query-miss` vào sổ → `schema-review` tính vào ngưỡng thêm type/page.

4. **Đổi layout**: tạo > 30 page cùng một type → `schema-review` đề xuất subfolder → duyệt → move file vào `wiki/<type>/`, rewrite link, kiểm resolve `[[slug]]` vẫn đúng sau split.

5. **File ngược qua inbox**: `knowhow-query` ra câu trả lời tốt, user OK file → kiểm item vào `inbox/` (KHÔNG vào thẳng `wiki/`), raw lưu Q&A tại `raw/YYYY-MM-DD-query-<slug>.md`.

6. **Reversibility**: sau một migrate, `git revert` → kiểm hệ về trạng thái cũ sạch.

### Tiêu chí pass v1.2

- distill/query ghi tín hiệu đúng định dạng vào `schema-signals.md`, KHÔNG tự đổi khuôn.
- `schema-review` chỉ đề xuất khi vượt ngưỡng, không báo nhiễu dưới ngưỡng.
- Migrate giữ link không hỏng, index rebuild đúng, SCHEMA version + changelog cập nhật.
- Bất biến "mọi knowhow qua inbox trước" không bị phá bởi query.
- Mọi migrate reversible bằng git.
- Log dùng prefix parse được: `grep "^## \[" .knowhow/wiki/log.md` ra danh sách op.
````

- [ ] **Step 3: Verify task.md cập nhật**

Run:
```bash
F=docs/superpowers/specs/task.md
grep -q "knowhow-query" "$F" && echo "query-row OK"
grep -q "Test end-to-end v1.2" "$F" && echo "scenarios OK"
grep -q "Reversibility" "$F" && echo "revert OK"
grep -q "Tiêu chí pass v1.2" "$F" && echo "criteria OK"
```
Expected: `query-row OK`, `scenarios OK`, `revert OK`, `criteria OK`.

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/specs/task.md
git commit -m "docs(v1.2): task.md thêm trạng thái query + 5 kịch bản kiểm chứng

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 10: Chạy end-to-end sandbox (5 kịch bản v1.2)

Mô phỏng toàn vòng đời v1.2 trên một sandbox, kiểm các tiêu chí cơ học bằng bash và phần phán đoán bằng đối chiếu skill. Đây là task verify cuối, không sửa skill (nếu phát hiện lỗi, quay lại task tương ứng).

**Files:**
- Không sửa file dự án. Chỉ tạo sandbox tạm để kiểm.

- [ ] **Step 1: Dựng sandbox + mô phỏng init**

Run:
```bash
SANDBOX=$(mktemp -d)/proj && mkdir -p "$SANDBOX" && cd "$SANDBOX"
mkdir -p .knowhow/{raw,inbox,archive,wiki,skills,workflows}
# SCHEMA living contract (rút gọn, đủ trường kiểm)
cat > .knowhow/SCHEMA.md <<'EOF'
# Knowhow Schema — Proj giả

**schema_version**: 1

## Page Types

| Type | Đường dẫn file | Mục đích |
|------|----------------|----------|
| decision | wiki/decision-<slug>.md | ... |
| pattern | wiki/pattern-<slug>.md | ... |
| concept | wiki/concept-<slug>.md | ... |
| troubleshooting | wiki/troubleshooting-<slug>.md | ... |

## Glossary & Convention

## Changelog

- 2026-06-05: Khởi tạo schema v1 (init).
EOF
cat > .knowhow/schema-signals.md <<'EOF'
# Schema Signals

## Đang chờ xử lý

## Đã xử lý
EOF
printf '# Activity Log\n\n## [2026-06-05] init | Khởi tạo .knowhow/\n' > .knowhow/wiki/log.md
# Kịch bản 1
grep -q "schema_version" .knowhow/SCHEMA.md && echo "K1 version OK"
grep -q "## Changelog" .knowhow/SCHEMA.md && echo "K1 changelog OK"
grep -q "## Glossary & Convention" .knowhow/SCHEMA.md && echo "K1 glossary OK"
test -f .knowhow/schema-signals.md && echo "K1 signals OK"
```
Expected: `K1 version OK`, `K1 changelog OK`, `K1 glossary OK`, `K1 signals OK`.

- [ ] **Step 2: Kịch bản 2 — distill emit no-fit-type ≥5, ngưỡng thêm type**

Run (tiếp trong `$SANDBOX`):
```bash
cd "$SANDBOX"
# mô phỏng 5 page experiment buộc thành concept + 5 tín hiệu no-fit-type
for i in 1 2 3 4 5; do
cat > .knowhow/wiki/concept-exp-$i.md <<EOF
---
type: concept
title: "Thí nghiệm $i"
tags: [experiment]
---
nội dung
EOF
echo "- [2026-06-05] distill | no-fit-type | item kết quả thí nghiệm $i | related: tag:experiment" >> .knowhow/schema-signals.md
done
# áp ngưỡng "thêm type": ≥5 no-fit-type cùng chủ đề
n=$(grep -c "no-fit-type" .knowhow/schema-signals.md)
[ "$n" -ge 5 ] && echo "K2 threshold-met OK ($n no-fit-type)"
c=$(grep -rh "^tags:" .knowhow/wiki | tr -d '[]' | tr ',' '\n' | grep -c experiment)
[ "$c" -ge 5 ] && echo "K2 tag-cluster OK ($c)"
```
Expected: `K2 threshold-met OK (5 no-fit-type)`, `K2 tag-cluster OK (5)`.

- [ ] **Step 3: Kịch bản 2 (tiếp) — mô phỏng migrate thêm type + bump + bước chung**

Run:
```bash
cd "$SANDBOX"
# thêm type vào SCHEMA Page Types
# (chèn dòng experiment vào bảng — dùng awk để chèn sau dòng troubleshooting)
awk '1; /troubleshooting \| wiki\/troubleshooting/ {print "| experiment | wiki/experiment-<slug>.md | Kết quả thí nghiệm |"}' .knowhow/SCHEMA.md > /tmp/s && mv /tmp/s .knowhow/SCHEMA.md
# reclassify 1 file (mô phỏng "từng file một")
git -C "$SANDBOX" init -q 2>/dev/null || true
mv .knowhow/wiki/concept-exp-1.md .knowhow/wiki/experiment-exp-1.md
# sửa type frontmatter
sed -i.bak 's/^type: concept/type: experiment/' .knowhow/wiki/experiment-exp-1.md && rm -f .knowhow/wiki/experiment-exp-1.md.bak
# bump version
sed -i.bak 's/\*\*schema_version\*\*: 1/**schema_version**: 2/' .knowhow/SCHEMA.md && rm -f .knowhow/SCHEMA.md.bak
# ghi changelog
sed -i.bak 's/- 2026-06-05: Khởi tạo schema v1 (init)./- 2026-06-06: Thêm type experiment — 5 page thí nghiệm.\n- 2026-06-05: Khởi tạo schema v1 (init)./' .knowhow/SCHEMA.md && rm -f .knowhow/SCHEMA.md.bak
# cắt tín hiệu sang "Đã xử lý"
# kiểm kết quả
grep -q "experiment | wiki/experiment-<slug>.md" .knowhow/SCHEMA.md && echo "K2 type-added OK"
grep -q "schema_version\*\*: 2" .knowhow/SCHEMA.md && echo "K2 bump OK"
grep -q "Thêm type experiment" .knowhow/SCHEMA.md && echo "K2 changelog OK"
test -f .knowhow/wiki/experiment-exp-1.md && echo "K2 naming OK"
grep -q "^type: experiment" .knowhow/wiki/experiment-exp-1.md && echo "K2 frontmatter OK"
```
Expected: `K2 type-added OK`, `K2 bump OK`, `K2 changelog OK`, `K2 naming OK`, `K2 frontmatter OK`.

- [ ] **Step 4: Kịch bản 3 — query emit query-miss**

Run:
```bash
cd "$SANDBOX"
echo '- [2026-06-07] query | query-miss | "so sánh 3 thí nghiệm A/B/C", phải chắp 4 page | related: tag:experiment' >> .knowhow/schema-signals.md
grep -qE '^- \[[0-9-]+\] query \| query-miss \| .+ \| related: ' .knowhow/schema-signals.md && echo "K3 query-miss-format OK"
```
Expected: `K3 query-miss-format OK`.

- [ ] **Step 5: Kịch bản 4 — đổi layout subfolder + resolve sau split**

Run:
```bash
cd "$SANDBOX"
mkdir -p .knowhow/wiki/experiment
# page trỏ link slug trần tới page sắp move
cat > .knowhow/wiki/decision-use-exp.md <<'EOF'
---
type: decision
title: "Dùng thí nghiệm"
---
Xem [[exp-1]].
EOF
# move page experiment vào subfolder
mv .knowhow/wiki/experiment-exp-1.md .knowhow/wiki/experiment/experiment-exp-1.md
# resolve [[exp-1]] đệ quy (type experiment trong TYPES)
hits=$(find .knowhow/wiki -name "experiment-exp-1.md" | wc -l | tr -d ' ')
[ "$hits" -eq 1 ] && echo "K4 resolve-after-split OK"
```
Expected: `K4 resolve-after-split OK`.

- [ ] **Step 6: Kịch bản 5 — file ngược qua inbox (không vào wiki)**

Run:
```bash
cd "$SANDBOX"
# mô phỏng query file ngược: raw + inbox, KHÔNG wiki
cat > .knowhow/raw/2026-06-07-query-so-sanh-exp.md <<'EOF'
# Q&A
Q: so sánh 3 thí nghiệm
A: ...
EOF
cat > .knowhow/inbox/2026-06-07-query-so-sanh-exp.md <<'EOF'
---
type: concept
title: "So sánh thí nghiệm A/B/C"
tags: [experiment]
captured_from: query
source_file: raw/2026-06-07-query-so-sanh-exp.md
---
## Tóm tắt
...
EOF
test -f .knowhow/inbox/2026-06-07-query-so-sanh-exp.md && echo "K5 inbox OK"
test -f .knowhow/raw/2026-06-07-query-so-sanh-exp.md && echo "K5 raw OK"
# đảm bảo KHÔNG có page query ghi thẳng wiki
! ls .knowhow/wiki/*query* >/dev/null 2>&1 && echo "K5 not-in-wiki OK"
grep -q "source_file: raw/2026-06-07-query" .knowhow/inbox/2026-06-07-query-so-sanh-exp.md && echo "K5 provenance OK"
```
Expected: `K5 inbox OK`, `K5 raw OK`, `K5 not-in-wiki OK`, `K5 provenance OK`.

- [ ] **Step 7: Kịch bản 6 — reversibility + log grep**

Run:
```bash
cd "$SANDBOX"
# log grep parse được
printf '## [2026-06-06] lint | Schema-review: thêm type experiment, 1 file migrate\n' >> .knowhow/wiki/log.md
grep "^## \[" .knowhow/wiki/log.md | tail -1 | grep -q "lint | Schema-review" && echo "K6 log-grep OK"
# reversibility: commit rồi revert (sandbox đã git init ở Step 3)
git -C "$SANDBOX" add -A && git -C "$SANDBOX" commit -qm "snapshot v1.2 migrate" 2>/dev/null
BEFORE=$(git -C "$SANDBOX" rev-parse HEAD)
# thực hiện một thay đổi rồi revert
echo "rác" >> .knowhow/SCHEMA.md
git -C "$SANDBOX" add -A && git -C "$SANDBOX" commit -qm "thay đổi thử"
git -C "$SANDBOX" revert --no-edit HEAD -q
! grep -q "^rác$" .knowhow/SCHEMA.md && echo "K6 revert OK"
```
Expected: `K6 log-grep OK`, `K6 revert OK`.

- [ ] **Step 8: Tổng kết + dọn sandbox**

Đối chiếu toàn bộ output K1-K6 đều `OK`. Nếu có FAIL, ghi lại kịch bản hỏng và quay về task tương ứng (K1→Task 1/2, K2→Task 4/6/8, K3→Task 5, K4→Task 7, K5→Task 5, K6→Task 3/6) để sửa.

Phần phán đoán LLM (distill phân loại đúng item, schema-review đề xuất đúng diff, query trích dẫn đúng) KHÔNG kiểm được bằng bash — đối chiếu bằng đọc lại Output format + ngưỡng trong các skill đã sửa, xác nhận chỉ dẫn rõ ràng và không mâu thuẫn.

Run (dọn):
```bash
echo "Sandbox: $SANDBOX (xoá thủ công nếu cần: rm -rf $(dirname "$SANDBOX"))"
```

- [ ] **Step 9: Commit (đánh dấu hoàn tất v1.2)**

Không có file dự án thay đổi ở task này. Tạo commit rỗng đánh dấu mốc kiểm chứng (tuỳ chọn):

```bash
git commit --allow-empty -m "test(v1.2): chạy 5 kịch bản sandbox end-to-end, pass

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Self-Review (đối chiếu plan với spec v1.2)

**Spec coverage** (mục "Có" + "Tiêu chí done cho v1.2"):

| Yêu cầu spec | Task phủ |
|---|---|
| SCHEMA.md living contract (schema_version + changelog) | Task 1, 2 |
| schema-signals.md accumulator (top-level, append-only, parse được) | Task 1 (mô tả), 2 (init tạo) |
| distill emit no-fit-type, adhoc-section | Task 4 |
| knowhow-query (trả lời + file ngược inbox + emit query-miss) | Task 5 |
| lint schema-review (đọc tín hiệu + quét sống + 4 ngưỡng + đề xuất) | Task 6 (reference), 8 (móc vào SKILL) |
| Migration 4 loại + rewrite link + rebuild index + bump version | Task 6 (playbook), 8 |
| resolve [[slug]] mở rộng type mới + subfolder (P0-B) | Task 7 |
| Log prefix parse được (L1) | Task 2 (init), 3 (capture/distill/lint), 5 (query) |
| 5 kịch bản kiểm chứng | Task 9 (viết), 10 (chạy) |

Không thấy gap. "Không có (để sau)" của spec (preset domain, search engine, auto-migrate, schema yaml, backfill tự động) đều KHÔNG được thêm — đúng phạm vi.

**Type consistency:** tên loại tín hiệu (`no-fit-type`, `adhoc-section`, `query-miss`), nguồn (`distill`, `query`), format dòng signal (`- [YYYY-MM-DD] <nguồn> | <loại> | <chi tiết> | related: <...>`), prefix log (`## [YYYY-MM-DD] <op> | <tiêu đề>`), đường dẫn `wiki/<type>-<slug>.md` nhất quán xuyên các task. Tập type đọc động từ SCHEMA (gọi `TYPES`) dùng nhất quán ở Task 7 và 8.

**Placeholder scan:** mọi step có nội dung markdown/bash thật. Không có "TBD"/"tương tự task N"/"thêm validation". Phần cần phán đoán LLM được nêu rõ là không bash-test được, kèm cách đối chiếu thay thế.
