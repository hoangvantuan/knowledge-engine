# Knowhow System Review & Fix v1.1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Sửa 2 bug đường dẫn, gia cố bước distill (lõi hệ thống), thêm metadata lifecycle, và 5 vá nhỏ để hệ thống knowhow dùng được thật.

**Architecture:** Hệ thống là 4 skill markdown thuần (`knowhow-init`, `knowhow-capture`, `knowhow-distill`, `knowhow-lint`) + reference docs. Không có code chạy được, không có test framework. Mỗi fix là chỉnh nội dung markdown. "Test" ở đây = lệnh `grep`/`ls` kiểm chứng trạng thái SAI hiện tại (đỏ) trước khi sửa, rồi kiểm chứng trạng thái ĐÚNG (xanh) sau khi sửa. Đây là cách áp TDD cho tài liệu/cấu hình.

**Tech Stack:** Markdown, YAML frontmatter, bash (grep/ls/mkdir), git.

**Nguồn:** Spec [specs/2026-06-05-knowhow-system-review-v1.1.md](../../../specs/2026-06-05-knowhow-system-review-v1.1.md) (đã vá G1, G2, A1-A5). Spec gốc [specs/2026-06-05-knowhow-system-design.md](../../../specs/2026-06-05-knowhow-system-design.md).

---

## File Structure

Bảng file bị động chạm và trách nhiệm sau khi sửa:

| File | Trách nhiệm | Task chạm |
|------|-------------|-----------|
| `skills/knowhow-init/SKILL.md` | Tạo cấu trúc `.knowhow/` phẳng, append config idempotent | 2, 11, 13, 14 |
| `skills/knowhow-init/references/schema-template.md` | Quy ước path phẳng, resolve slug, lifecycle, onboarding đa agent | 2, 4, 9, 12, 13 |
| `skills/knowhow-init/references/agent-config-snippet.md` | Forcing function nhắc distill | 7 |
| `skills/knowhow-capture/SKILL.md` | Luôn lưu raw, ghi `source_file` trỏ raw/ | 8 |
| `skills/knowhow-capture/references/page-formats.md` | Path phẳng, `source_file` bắt buộc, `status`+`confidence` mọi type | 2, 8, 9 |
| `skills/knowhow-distill/SKILL.md` | Grep nội dung, path tương đối, changelog trỏ raw/, set lifecycle, rewrite inbound link | 3, 6, 8, 9, 10, 14 |
| `skills/knowhow-lint/SKILL.md` | Resolve slug, bắt `status`, hạ confidence, rebuild-index | 4, 9, 12 |
| `specs/2026-06-05-knowhow-system-design.md` | Bỏ "tự cải tiến" | 10 |
| `specs/task.md` | Kịch bản test end-to-end (file mới) | 15 |

**Thứ tự thực thi** bám priority của spec: P0 (Task 1-4) → P1-C (Task 5-9) → P1-D (Task 10) → P2 (Task 11-14) → e2e (Task 15).

---

## Task 1: Khởi tạo git để enable commit workflow

**Files:**
- Create: `.gitignore`

Repo hiện chưa phải git repository. Plan dùng frequent commits nên cần init trước.

- [ ] **Step 1: Kiểm chứng chưa có git**

Run: `git -C /Users/tuanhv/Desktop/knowledge-engine rev-parse --is-inside-work-tree 2>&1`
Expected: `fatal: not a git repository` (hoặc tương đương)

- [ ] **Step 2: Init git và tạo .gitignore**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git init
git branch -M main
```

Tạo `.gitignore`:
```
.DS_Store
*.log
```

- [ ] **Step 3: Verify git hoạt động**

Run: `git -C /Users/tuanhv/Desktop/knowledge-engine status --short`
Expected: liệt kê các file chưa track (CLAUDE.md, skills/, specs/, docs/)

- [ ] **Step 4: Commit baseline**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add -A
git commit -m "chore: init repo + baseline trước fix v1.1"
```

---

## Task 2: P0-A — Chốt một quy ước đường dẫn wiki phẳng

**Files:**
- Modify: `skills/knowhow-init/SKILL.md` (Bước 2: cây thư mục + lệnh mkdir)
- Modify: `skills/knowhow-init/references/schema-template.md` (bảng Page Types + Naming Conventions)
- Modify: `skills/knowhow-capture/references/page-formats.md` (đảm bảo header nhất quán — đã đúng, chỉ xác nhận)

Quyết định: `wiki/<type>-<slug>.md` phẳng. Bỏ 4 subfolder.

- [ ] **Step 1: Kiểm chứng bug đang tồn tại**

Run: `grep -n "decisions/\|patterns/\|concepts/\|troubleshooting/" skills/knowhow-init/SKILL.md skills/knowhow-init/references/schema-template.md`
Expected: có hit ở init SKILL (cây thư mục + mkdir) và schema-template (bảng Page Types) — chứng tỏ quy ước subfolder vẫn còn.

- [ ] **Step 2: Sửa cây thư mục trong init SKILL.md Bước 2**

Trong [skills/knowhow-init/SKILL.md](../../../skills/knowhow-init/SKILL.md), thay block cây thư mục (đoạn ```` ``` ```` mô tả `.knowhow/`) thành:

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

(Lưu ý: đã thêm `archive/` cho Task 13, đã bỏ 4 subfolder wiki.)

- [ ] **Step 3: Sửa lệnh mkdir trong init SKILL.md Bước 2**

Thay dòng lệnh mkdir cũ thành:

```bash
mkdir -p .knowhow/{raw,inbox,archive,wiki,skills,workflows}
```

- [ ] **Step 4: Sửa bảng Page Types trong schema-template.md**

Trong [skills/knowhow-init/references/schema-template.md](../../../skills/knowhow-init/references/schema-template.md), thay bảng Page Types thành (cột Thư mục đều `wiki/`):

```markdown
## Page Types

| Type | Đường dẫn file | Mục đích |
|------|----------------|----------|
| decision | wiki/decision-<slug>.md | Quyết định + lý do + bối cảnh |
| pattern | wiki/pattern-<slug>.md | Cách làm đã chứng minh hiệu quả |
| concept | wiki/concept-<slug>.md | Thuật ngữ, khái niệm riêng dự án |
| troubleshooting | wiki/troubleshooting-<slug>.md | Sự cố đã gặp + cách xử lý |
| skill | skills/<slug>.md | Công việc cụ thể, chạy độc lập, tái sử dụng |
| workflow | workflows/<slug>.md | Chuỗi bước, gắn domain, gọi nhiều skill |
```

- [ ] **Step 5: Sửa Naming Conventions trong schema-template.md**

Thay mục Naming Conventions thành:

```markdown
## Naming Conventions

- File wiki: `wiki/<type>-<slug>.md`, slug `kebab-case`. Ví dụ: `wiki/decision-rest-to-graphql.md`, `wiki/pattern-retry-with-jitter.md`.
- Skill: `skills/<slug>.md`, slug động từ + danh từ (`skills/parse-invoice.md`, `skills/write-commit-msg.md`).
- Workflow: `workflows/<slug>.md`, slug danh từ mô tả quy trình (`workflows/release-checklist.md`).
```

- [ ] **Step 6: Xác nhận page-formats.md đã đúng (không cần sửa header)**

Run: `grep -n "^File: \`wiki/" skills/knowhow-capture/references/page-formats.md`
Expected: 4 dòng `wiki/decision-slug.md`, `wiki/pattern-slug.md`, `wiki/concept-slug.md`, `wiki/troubleshooting-slug.md` — đã phẳng, đúng.

- [ ] **Step 7: Verify quy ước đã nhất quán**

Run: `grep -rn "wiki/decisions/\|wiki/patterns/\|wiki/concepts/\|wiki/troubleshooting/" skills/`
Expected: KHÔNG còn hit nào (rỗng). Mọi tham chiếu subfolder đã biến mất.

- [ ] **Step 8: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-init/SKILL.md skills/knowhow-init/references/schema-template.md
git commit -m "fix(P0-A): chốt quy ước wiki phẳng wiki/<type>-<slug>.md, bỏ subfolder"
```

---

## Task 3: P0-A2 (G1) — Sửa hardcode path Gemini trong distill

**Files:**
- Modify: `skills/knowhow-distill/SKILL.md` (Bước 5, mục 1: Format reference)

Bug cùng họ P0-A: distill hardcode đường dẫn tuyệt đối gắn Gemini, gãy khi deploy chỗ khác.

- [ ] **Step 1: Kiểm chứng bug**

Run: `grep -n "gemini" skills/knowhow-distill/SKILL.md`
Expected: 1 hit dòng ~78 `~/.gemini/config/skills/knowhow-capture/references/page-formats.md`

- [ ] **Step 2: Sửa thành đường dẫn tương đối**

Trong [skills/knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md) Bước 5 mục 1, thay dòng:

```
   - Format reference: `~/.gemini/config/skills/knowhow-capture/references/page-formats.md`
```

thành:

```
   - Format reference: `../knowhow-capture/references/page-formats.md` (đường dẫn tương đối từ thư mục skill này). Nếu không resolve được, dùng format frontmatter cơ bản bên dưới.
```

- [ ] **Step 3: Verify không còn hardcode path tuyệt đối**

Run: `grep -rn "gemini\|~/.claude/config" skills/`
Expected: KHÔNG còn hit nào (rỗng).

- [ ] **Step 4: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-distill/SKILL.md
git commit -m "fix(G1): distill reference page-formats.md bằng path tương đối, bỏ hardcode Gemini"
```

---

## Task 4: P0-B (A5) — Thuật toán resolve `[[slug]]` match chặt type-slug

**Files:**
- Modify: `skills/knowhow-lint/SKILL.md` (mục 1b Link integrity)
- Modify: `skills/knowhow-init/references/schema-template.md` (mục Cross-referencing)

- [ ] **Step 1: Kiểm chứng lint 1b chưa định nghĩa resolve**

Run: `grep -n "Resolve link thành file path" skills/knowhow-lint/SKILL.md`
Expected: 1 hit, dòng đứng trơ không có thuật toán theo sau.

- [ ] **Step 2: Thay mục 1b trong lint SKILL.md bằng thuật toán đầy đủ**

Trong [skills/knowhow-lint/SKILL.md](../../../skills/knowhow-lint/SKILL.md), thay nội dung mục `### 1b. Link integrity` thành:

```markdown
### 1b. Link integrity

- Tìm tất cả `[[...]]` trong body các page (wiki, skills, workflows).
- Resolve mỗi link `[[X]]` theo thuật toán (match CHẶT type-slug, không glob đuôi tự do):
  1. Nếu `X` có dạng `<type>-<slug>` với type ∈ {decision, pattern, concept, troubleshooting}: khớp chính xác `wiki/<type>-<slug>.md`. Dạng tường minh, không ambiguous.
  2. Nếu `X` là slug trần: thử khớp chính xác từng ứng viên `wiki/decision-X.md`, `wiki/pattern-X.md`, `wiki/concept-X.md`, `wiki/troubleshooting-X.md`. Đếm số file tồn tại:
     - Đúng 1 → resolve OK.
     - 0 → báo link hỏng (ghi page nguồn + target).
     - ≥2 (cùng slug khác type) → báo **ambiguous**, yêu cầu link kèm type `[[decision-X]]`.
  3. Skill/workflow: khớp chính xác `skills/X.md` hoặc `workflows/X.md`.
- File không tồn tại → báo link hỏng, ghi rõ page nguồn và target.
```

- [ ] **Step 3: Thêm cú pháp slug vào Cross-referencing của schema-template.md**

Trong [skills/knowhow-init/references/schema-template.md](../../../skills/knowhow-init/references/schema-template.md), thay mục Cross-referencing thành:

```markdown
## Cross-referencing

- Dùng `[[slug]]` để liên kết giữa các page. `slug` = phần sau prefix type. Ví dụ file `wiki/decision-rest-to-graphql.md` có slug `rest-to-graphql`, link bằng `[[rest-to-graphql]]`.
- Nếu nhiều page khác type trùng slug, link phải kèm type để khử nhập nhằng: `[[decision-rest-to-graphql]]`.
- Workflow reference skill bằng `→ Dùng skill: [[skill-name]]`.
- Mọi page phải xuất hiện trong index.md hoặc registry.md tương ứng.
```

- [ ] **Step 4: Verify thuật toán đã có mặt**

Run: `grep -n "ambiguous\|match CHẶT\|khớp chính xác" skills/knowhow-lint/SKILL.md`
Expected: ≥2 hit — thuật toán đã được nhúng.

- [ ] **Step 5: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-lint/SKILL.md skills/knowhow-init/references/schema-template.md
git commit -m "fix(P0-B): định nghĩa thuật toán resolve [[slug]] match chặt type-slug"
```

---

## Task 5: P1-C1 — Grep nội dung trước khi distill quyết định

**Files:**
- Modify: `skills/knowhow-distill/SKILL.md` (Bước 3: Phân tích và đề xuất)

Lõi hệ thống: distill hiện "so sánh nội dung với registry" nhưng registry chỉ có title. Chèn grep nội dung thật.

- [ ] **Step 1: Kiểm chứng lỗ hổng mù nội dung**

Run: `grep -n "So sánh title, tags, nội dung với registry" skills/knowhow-distill/SKILL.md`
Expected: 1 hit — yêu cầu so sánh nội dung nhưng không có cơ chế grep.

- [ ] **Step 2: Thay mục 2 trong Bước 3 distill bằng bước grep**

Trong [skills/knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md) Bước 3, thay mục `2. **Tìm trùng**: ...` thành:

```markdown
2. **Tìm trùng (grep nội dung thật, chống mù)**: Registry chỉ có title + mô tả 1 dòng, KHÔNG đủ để so sánh nội dung. Với mỗi inbox item:
   - Rút 3-5 từ khoá từ tiêu đề + `tags` của item.
   - Chạy grep tìm page liên quan:
     ```bash
     grep -ril "<từ khoá>" .knowhow/wiki .knowhow/skills .knowhow/workflows
     ```
   - Đọc các file hit. CHỈ sau khi đọc nội dung thật mới áp bảng quyết định (tạo mới / cập nhật / sửa / refine / gộp / bỏ qua).
   - Lưới an toàn: `knowhow-lint consolidation` chạy định kỳ bắt trùng mà grep lọt.
```

- [ ] **Step 3: Bổ sung ghi chú tag bắt buộc**

Ngay dưới mục vừa thêm (vẫn trong Bước 3), thêm dòng:

```markdown
> **Lưu ý**: Capture phải gán `tags` nhất quán để grep theo tag hiệu quả. Item không tag → grep chỉ dựa từ khoá tiêu đề, dễ lọt trùng.
```

- [ ] **Step 4: Verify grep đã được nhúng**

Run: `grep -n "grep -ril" skills/knowhow-distill/SKILL.md`
Expected: 1 hit — lệnh grep nội dung đã có trong flow distill.

- [ ] **Step 5: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-distill/SKILL.md
git commit -m "feat(C1): distill grep nội dung thật trước khi quyết định, chống mù nội dung"
```

---

## Task 6: P1-C2 — Forcing function nhắc chạy distill

**Files:**
- Modify: `skills/knowhow-init/references/agent-config-snippet.md`
- Modify: `skills/knowhow-capture/SKILL.md` (Bước 6: gợi ý + cờ `--then-distill`)

- [ ] **Step 1: Kiểm chứng chưa có forcing function**

Run: `grep -ni "5 item\|7 ngày\|then-distill" skills/knowhow-init/references/agent-config-snippet.md skills/knowhow-capture/SKILL.md`
Expected: KHÔNG hit — chưa có lực đẩy nào.

- [ ] **Step 2: Thêm dòng nhắc vào agent-config-snippet.md**

Trong [skills/knowhow-init/references/agent-config-snippet.md](../../../skills/knowhow-init/references/agent-config-snippet.md), thêm mục 5 vào cuối danh sách:

```markdown
5. Khi `.knowhow/inbox/` có ≥ 5 item hoặc có item cũ hơn 7 ngày, chủ động nhắc user chạy `knowhow-distill` để đúc kết, tránh inbox tồn đọng.
```

- [ ] **Step 3: Thêm cờ `--then-distill` vào capture Bước 6**

Trong [skills/knowhow-capture/SKILL.md](../../../skills/knowhow-capture/SKILL.md) Bước 6, thay đoạn gợi ý cuối thành:

```markdown
Gợi ý chạy `knowhow-distill` để đúc kết inbox thành wiki/skill/workflow.

**Chế độ liền mạch (user solo)**: Nếu user gọi capture kèm cờ `--then-distill`, sau khi ghi inbox xong, chạy luôn `knowhow-distill` trong cùng phiên để duyệt + đúc kết liền 2 bước, không phải gọi lại.
```

- [ ] **Step 4: Verify forcing function đã có**

Run: `grep -ni "≥ 5 item\|then-distill" skills/knowhow-init/references/agent-config-snippet.md skills/knowhow-capture/SKILL.md`
Expected: 2 hit — cả nhắc tự động lẫn cờ liền mạch.

- [ ] **Step 5: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-init/references/agent-config-snippet.md skills/knowhow-capture/SKILL.md
git commit -m "feat(C2): forcing function nhắc distill + cờ --then-distill"
```

---

## Task 7: P1-C3 (A1) — Provenance sống sót sau distill

**Files:**
- Modify: `skills/knowhow-capture/SKILL.md` (Bước 4: luôn lưu raw)
- Modify: `skills/knowhow-capture/references/page-formats.md` (mô tả `source_file` bắt buộc)
- Modify: `skills/knowhow-distill/SKILL.md` (Bước 5: changelog + log trỏ raw/)

A1 đã chốt: dùng field `source_file` (đã có), không đẻ field `source:` mới.

- [ ] **Step 1: Kiểm chứng provenance đứt**

Run: `grep -n "Nếu có file nguồn gốc, copy vào" skills/knowhow-capture/SKILL.md; grep -n "source: inbox" skills/knowhow-distill/SKILL.md`
Expected: capture chỉ copy raw "nếu có file ngoài"; distill changelog mẫu trỏ `inbox/...` (sẽ bị xoá → trace chết).

- [ ] **Step 2: Sửa capture Bước 4 — luôn lưu raw**

Trong [skills/knowhow-capture/SKILL.md](../../../skills/knowhow-capture/SKILL.md) Bước 4, thay mục 2 (`Nếu có file nguồn gốc...`) thành:

```markdown
2. **Luôn lưu nguồn vào `raw/`** (cả khi nguồn là hội thoại, không có file ngoài):
   - Ghi trích đoạn nguyên văn nguồn vào `raw/YYYY-MM-DD-slug.md` (cùng slug với inbox item).
   - Set frontmatter inbox `source_file: raw/YYYY-MM-DD-slug.md` (đường dẫn tương đối từ `.knowhow/`). KHÔNG để trống. KHÔNG trỏ vào chính file inbox.
```

- [ ] **Step 3: Sửa mô tả `source_file` trong page-formats.md**

Trong [skills/knowhow-capture/references/page-formats.md](../../../skills/knowhow-capture/references/page-formats.md), mục 1 Inbox Item Format, sửa dòng comment frontmatter:

Thay:
```yaml
source_file: ""  # Đường dẫn file gốc trong raw/ (nếu có)
```
thành:
```yaml
source_file: "raw/YYYY-MM-DD-slug.md"  # BẮT BUỘC. Trích đoạn nguồn (cả hội thoại) luôn lưu vào raw/. Không để trống.
```

Cũng cập nhật ví dụ inbox item loại decision (frontmatter của ví dụ) thêm dòng `source_file: "raw/2026-06-05-chon-postgresql.md"`.

- [ ] **Step 4: Sửa distill Bước 5 — changelog + log trỏ raw/**

Trong [skills/knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md) Bước 5 mục 4 (Changelog), thay mẫu thành:

```markdown
4. **Changelog**: Ghi dòng changelog cuối page bị thay đổi. Lấy nguồn từ `source_file` của inbox item (trỏ `raw/...`), KHÔNG trỏ `inbox/...` (inbox sẽ bị xoá ở mục 7).
   ```
   ## Changelog
   - YYYY-MM-DD: [mô tả thay đổi] (source: raw/YYYY-MM-DD-slug.md)
   ```
```

- [ ] **Step 5: Verify provenance trỏ raw/**

Run: `grep -n "source: raw\|Luôn lưu nguồn vào" skills/knowhow-distill/SKILL.md skills/knowhow-capture/SKILL.md`
Expected: 2 hit — capture luôn lưu raw, distill changelog trỏ raw/.

Run: `grep -rn "source: inbox" skills/`
Expected: KHÔNG hit — không còn changelog trỏ inbox.

- [ ] **Step 6: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-capture/SKILL.md skills/knowhow-capture/references/page-formats.md skills/knowhow-distill/SKILL.md
git commit -m "fix(C3): provenance luôn lưu raw, source_file trỏ raw/, changelog không trỏ inbox"
```

---

## Task 8: P1-C4 (A2, A3) — Metadata sống: status + confidence lifecycle

**Files:**
- Modify: `skills/knowhow-init/references/schema-template.md` (định nghĩa lifecycle)
- Modify: `skills/knowhow-capture/references/page-formats.md` (thêm field vào frontmatter mọi type)
- Modify: `skills/knowhow-distill/SKILL.md` (set/nâng confidence + status)

A2: `status` mọi page gồm skill/workflow. A3: confidence đếm theo số entry changelog.

- [ ] **Step 1: Kiểm chứng field thiếu**

Run: `grep -c "status:" skills/knowhow-capture/references/page-formats.md; grep -n "confidence" skills/knowhow-capture/references/page-formats.md | head`
Expected: `status:` đếm = 0 (chưa có ở đâu); `confidence` có ở decision/pattern/troubleshooting nhưng KHÔNG có ở concept.

- [ ] **Step 2: Thêm `status` vào frontmatter mọi type trong page-formats.md**

Trong [skills/knowhow-capture/references/page-formats.md](../../../skills/knowhow-capture/references/page-formats.md), thêm dòng `status: active` vào MỌI block frontmatter của 6 type (decision, pattern, concept, troubleshooting, skill, workflow), đặt ngay dưới `updated:`. Mẫu cho mỗi block:

```yaml
status: active   # active | deprecated | archived
```

Áp cho: 2.1 Decision, 2.2 Pattern, 2.3 Concept, 2.4 Troubleshooting, 3 Skill, 4 Workflow (cả frontmatter mẫu lẫn frontmatter trong ví dụ).

- [ ] **Step 3: Thêm `confidence` vào concept trong page-formats.md**

Trong block frontmatter của 2.3 Concept (cả mẫu lẫn ví dụ), thêm dòng dưới `updated:`:

```yaml
confidence: low | medium | high
```

(Ví dụ concept set `confidence: medium`.)

- [ ] **Step 4: Định nghĩa lifecycle trong schema-template.md**

Trong [skills/knowhow-init/references/schema-template.md](../../../skills/knowhow-init/references/schema-template.md), thêm mục mới ngay sau Cross-referencing:

```markdown
## Vòng đời metadata

### status (mọi page, gồm skill/workflow)

- `active`: đang dùng. Mặc định khi tạo.
- `deprecated`: còn để tham khảo nhưng có cách mới tốt hơn. Distill set khi thay thế cách cũ.
- `archived`: lỗi thời, chuyển vào `archive/`. Lint consolidation set.

### confidence (chỉ 4 wiki type, skill/workflow dùng version)

Đếm theo **số entry trong phần Changelog** của page:
- 1 entry (mới tạo) → `low`.
- ≥2 entry → `medium`.
- ≥3 entry → `high`.

- capture set `low` cho item mới.
- distill nâng khi page được cập nhật lặp lại (grep trỏ về page cũ → CẬP NHẬT → thêm entry changelog).
- lint consolidation hạ 1 bậc khi `updated` cũ hơn 90 ngày và không có entry changelog mới trong khoảng đó.
```

- [ ] **Step 5: Sửa distill set lifecycle**

Trong [skills/knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md) Bước 5, thêm mục mới sau mục 5 (Version):

```markdown
5b. **Lifecycle metadata**:
   - `status`: page mới set `active`. Khi hành động là SỬA (thay cách cũ), set page/section cũ `status: deprecated`. Khi GỘP, page bị nuốt set `status: archived`.
   - `confidence` (chỉ wiki, không áp skill/workflow): set theo số entry Changelog sau khi ghi entry mới — 1 → `low`, ≥2 → `medium`, ≥3 → `high`.
```

Đồng thời trong **Bảng quyết định hành động** (đầu file), sửa ô hành động "SỬA" từ `đánh dấu cách cũ deprecated` thành `set status: deprecated cho cách cũ`.

- [ ] **Step 6: Verify field + lifecycle có mặt**

Run: `grep -c "status: active" skills/knowhow-capture/references/page-formats.md`
Expected: ≥6 (mỗi type ≥1, có ví dụ thì hơn).

Run: `grep -n "Vòng đời metadata\|≥3 entry" skills/knowhow-init/references/schema-template.md`
Expected: có hit — lifecycle định nghĩa xong.

- [ ] **Step 7: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-init/references/schema-template.md skills/knowhow-capture/references/page-formats.md skills/knowhow-distill/SKILL.md
git commit -m "feat(C4): thêm status mọi page + confidence concept + lifecycle đếm theo changelog"
```

---

## Task 9: P1-D — Bỏ chữ "tự cải tiến" cho trung thực (Chánh Ngữ)

**Files:**
- Modify: `specs/2026-06-05-knowhow-system-design.md` (dòng mục tiêu + heading)
- Modify: `skills/knowhow-init/SKILL.md` (Tổng quan)
- Modify: `skills/knowhow-distill/SKILL.md` (lý do QUY TẮC SỐ 1)

- [ ] **Step 1: Grep mọi chỗ "tự cải tiến"**

Run: `grep -rn "tự cải tiến" specs/ skills/`
Expected: hit ở design.md (dòng mục tiêu + heading "Vòng đời tự cải tiến"), init SKILL.md Tổng quan, distill SKILL.md.

- [ ] **Step 2: Sửa design.md dòng mục tiêu**

Trong [specs/2026-06-05-knowhow-system-design.md](../../../specs/2026-06-05-knowhow-system-design.md), thay cụm trong dòng `**Mục tiêu**:`:

Thay `biến mỗi dự án thành "bộ não" tự cải tiến.`
thành `biến mỗi dự án thành kho tri thức tích luỹ có cấu trúc, AI viết và người duyệt.`

- [ ] **Step 3: Sửa heading design.md**

Thay `## Vòng đời tự cải tiến`
thành `## Vòng đời tích luỹ tri thức`

- [ ] **Step 4: Sửa init SKILL.md Tổng quan**

Trong [skills/knowhow-init/SKILL.md](../../../skills/knowhow-init/SKILL.md), thay:

`Knowhow System biến mỗi dự án thành "bộ não" tự cải tiến qua 3 lớp:`
thành
`Knowhow System là nghi thức tích luỹ tri thức có cấu trúc cho dự án (AI viết, người duyệt) qua 3 lớp:`

- [ ] **Step 5: Sửa distill SKILL.md**

Trong [skills/knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md), thay:

`Hệ thống knowhow tự cải tiến bằng cách tích lũy.`
thành
`Hệ thống knowhow tích luỹ giá trị bằng cách gộp tri thức, không phân mảnh.`

- [ ] **Step 6: Verify sạch**

Run: `grep -rn "tự cải tiến" specs/ skills/`
Expected: KHÔNG hit (rỗng).

- [ ] **Step 7: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add specs/2026-06-05-knowhow-system-design.md skills/knowhow-init/SKILL.md skills/knowhow-distill/SKILL.md
git commit -m "docs(D): bỏ 'tự cải tiến', đổi sang 'tích luỹ AI viết người duyệt' cho trung thực"
```

---

## Task 10: P2-E, P2-G — Onboarding đa agent + rewrite inbound link khi gộp/deprecate

**Files:**
- Modify: `skills/knowhow-init/references/schema-template.md` (mục Onboarding)
- Modify: `skills/knowhow-distill/SKILL.md` (Bước 5: gộp/deprecate)

- [ ] **Step 1: Kiểm chứng thiếu**

Run: `grep -ni "chỉ đọc\|Antigravity\|inbound\|\[\[old" skills/knowhow-init/references/schema-template.md skills/knowhow-distill/SKILL.md`
Expected: KHÔNG hit — chưa có ghi chú giới hạn agent, chưa có rewrite inbound link.

- [ ] **Step 2: Thêm ghi chú đa agent vào Onboarding (E)**

Trong [skills/knowhow-init/references/schema-template.md](../../../skills/knowhow-init/references/schema-template.md), cuối mục Onboarding, thêm:

```markdown
> **Phạm vi agent**: `.knowhow/` là markdown thuần, MỌI agent ĐỌC được. Nhưng 4 skill vận hành (init, capture, distill, lint) hiện viết cho Antigravity (Gemini). Agent khác (Claude Code, Codex) chỉ ĐỌC knowhow, chưa chạy được capture/distill/lint cho tới khi skill được port.
```

- [ ] **Step 3: Thêm rewrite inbound link khi gộp/deprecate (G)**

Trong [skills/knowhow-distill/SKILL.md](../../../skills/knowhow-distill/SKILL.md) Bước 5, thêm mục mới sau mục 3 (Cross-referencing):

```markdown
3b. **Rewrite inbound link khi GỘP/deprecate**: Khi gộp page hoặc set deprecated, page khác có thể đang trỏ `[[old-slug]]`. Grep toàn repo và sửa:
   ```bash
   grep -rln "\[\[old-slug\]\]" .knowhow
   ```
   - GỘP: đổi `[[old-slug]]` → `[[new-slug]]` ở mọi file nguồn.
   - Deprecate (vẫn giữ page): để link nguyên nhưng đảm bảo page đích có `status: deprecated` để người đọc biết.
```

- [ ] **Step 4: Verify**

Run: `grep -ni "chỉ ĐỌC knowhow\|Rewrite inbound link" skills/knowhow-init/references/schema-template.md skills/knowhow-distill/SKILL.md`
Expected: 2 hit.

- [ ] **Step 5: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-init/references/schema-template.md skills/knowhow-distill/SKILL.md
git commit -m "fix(E,G): ghi chú phạm vi đa agent + rewrite inbound link khi gộp/deprecate"
```

---

## Task 11: P2-F (A4) — Lệnh rebuild-index cho lint

**Files:**
- Modify: `skills/knowhow-lint/SKILL.md` (thêm chế độ rebuild-index)

A4: rebuild quét frontmatter mọi file trong wiki/, skills/, workflows/ → sinh lại cả 3 file dẫn xuất.

- [ ] **Step 1: Kiểm chứng chưa có rebuild**

Run: `grep -ni "rebuild" skills/knowhow-lint/SKILL.md`
Expected: KHÔNG hit.

- [ ] **Step 2: Thêm chế độ rebuild-index vào lint**

Trong [skills/knowhow-lint/SKILL.md](../../../skills/knowhow-lint/SKILL.md), thêm mục `### 1f. Mode đặc biệt: rebuild-index` vào cuối Chế độ 1 (trước phần Output format), và thêm trigger ở mục Chọn chế độ:

Tại mục "Chọn chế độ", thêm dòng:
```markdown
- User nói "rebuild", "rebuild-index", "sinh lại index" → chạy **Rebuild Index**.
```

Thêm section mới (đặt sau Chế độ 2 Consolidation):
```markdown
---

## Chế độ 3: Rebuild Index (file dẫn xuất)

`wiki/index.md`, `skills/registry.md`, `workflows/registry.md` là **file dẫn xuất** từ frontmatter các page. Khi merge conflict ở 3 file này, không cần giải tay — sinh lại.

### Bước thực hiện

1. Quét frontmatter mọi file trong `wiki/` (trừ index.md, log.md): đọc `type`, `title`, `tags`, `updated`.
2. Sinh `wiki/index.md`, group theo `type` (4 heading: Decisions, Patterns, Concepts, Troubleshooting), mỗi dòng `- [[<type>-<slug>]] - <title>`.
3. Quét frontmatter mọi file trong `skills/` (trừ registry.md): đọc `title`, `version`, `tags`, `updated`. Sinh `skills/registry.md` theo format page-formats mục 6.1, sort alphabet.
4. Quét frontmatter mọi file trong `workflows/` (trừ registry.md): đọc `title`, `skills_used`, `version`, `updated`. Sinh `workflows/registry.md` theo format mục 6.2, sort alphabet.
5. Ghi log: `- [lint] Rebuild index + 2 registry từ frontmatter`.
```

- [ ] **Step 3: Verify**

Run: `grep -ni "Rebuild Index\|file dẫn xuất" skills/knowhow-lint/SKILL.md`
Expected: ≥2 hit.

- [ ] **Step 4: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-lint/SKILL.md
git commit -m "feat(F): thêm chế độ rebuild-index sinh lại index + 2 registry từ frontmatter"
```

---

## Task 12: P2-H — Tạo `archive/` trong init + mô tả trong SCHEMA

**Files:**
- Modify: `skills/knowhow-init/references/schema-template.md` (bảng Cấu trúc thư mục)
- (init SKILL.md cây thư mục + mkdir đã thêm `archive/` ở Task 2)

- [ ] **Step 1: Kiểm chứng archive thiếu mô tả**

Run: `grep -n "archive" skills/knowhow-init/references/schema-template.md`
Expected: KHÔNG hit — SCHEMA chưa nhắc archive.

Run: `grep -n "archive" skills/knowhow-init/SKILL.md`
Expected: CÓ hit (Task 2 đã thêm vào cây + mkdir). Nếu chưa, quay lại Task 2 Step 2-3.

- [ ] **Step 2: Thêm archive/ vào bảng Cấu trúc thư mục của SCHEMA**

Trong [skills/knowhow-init/references/schema-template.md](../../../skills/knowhow-init/references/schema-template.md), thêm dòng vào bảng "Cấu trúc thư mục" (sau dòng inbox/):

```markdown
| archive/ | Page lỗi thời đã rút khỏi wiki. Lint consolidation chuyển vào đây. |
```

- [ ] **Step 3: Verify**

Run: `grep -n "archive/" skills/knowhow-init/references/schema-template.md`
Expected: 1 hit — đã mô tả.

- [ ] **Step 4: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-init/references/schema-template.md
git commit -m "fix(H): mô tả archive/ trong SCHEMA (init đã tạo folder ở Task 2)"
```

---

## Task 13: P2-I — Append snippet idempotent trong init

**Files:**
- Modify: `skills/knowhow-init/SKILL.md` (Bước 5)

- [ ] **Step 1: Kiểm chứng không idempotent**

Run: `grep -n "## Knowhow" skills/knowhow-init/SKILL.md`
Expected: KHÔNG hit — Bước 5 chưa check trùng trước khi append.

- [ ] **Step 2: Sửa init Bước 5 thêm guard idempotent**

Trong [skills/knowhow-init/SKILL.md](../../../skills/knowhow-init/SKILL.md) Bước 5, thay mục "Quy tắc" thành:

```markdown
Quy tắc:
- Thêm vào file **đầu tiên** tìm thấy.
- Nếu **chưa có file nào**, tạo `CLAUDE.md` mới.
- **Idempotent**: TRƯỚC khi append, grep `## Knowhow` trong file config. Nếu đã có → BỎ QUA, không append lần hai (tránh nhân đôi khi init chạy lại):
  ```bash
  grep -q "^## Knowhow" <config-file> && echo "Đã có, bỏ qua" || cat references/agent-config-snippet.md >> <config-file>
  ```
- Nội dung thêm: đọc `references/agent-config-snippet.md` và append vào cuối file.
```

- [ ] **Step 3: Verify guard có mặt**

Run: `grep -n "Idempotent\|grep -q" skills/knowhow-init/SKILL.md`
Expected: 2 hit.

- [ ] **Step 4: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add skills/knowhow-init/SKILL.md
git commit -m "fix(I): init Bước 5 idempotent, grep ## Knowhow trước khi append config"
```

---

## Task 14: G2 — Tạo `specs/task.md` với kịch bản test end-to-end

**Files:**
- Create: `specs/task.md`

Spec trỏ `task.md` nhưng file không tồn tại. Tạo file mô tả trạng thái 4 skill + kịch bản e2e để verify toàn bộ fix v1.1.

- [ ] **Step 1: Kiểm chứng file thiếu**

Run: `ls specs/task.md 2>&1`
Expected: `No such file or directory`

- [ ] **Step 2: Tạo specs/task.md**

Tạo [specs/task.md](../../../specs/task.md) với nội dung:

```markdown
# Task: Knowhow System

## Trạng thái implement

| Skill | Trạng thái | File |
|-------|-----------|------|
| knowhow-init | Done (v1.1) | skills/knowhow-init/SKILL.md |
| knowhow-capture | Done (v1.1) | skills/knowhow-capture/SKILL.md |
| knowhow-distill | Done (v1.1) | skills/knowhow-distill/SKILL.md |
| knowhow-lint | Done (v1.1) | skills/knowhow-lint/SKILL.md |

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
```

- [ ] **Step 3: Verify file tồn tại**

Run: `ls -la specs/task.md && grep -c "knowhow-" specs/task.md`
Expected: file tồn tại, grep ≥4.

- [ ] **Step 4: Commit**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add specs/task.md
git commit -m "docs(G2): tạo task.md với trạng thái 4 skill + kịch bản e2e test v1.1"
```

---

## Task 15: Verify toàn cục — đối chiếu Tiêu chí done v1.1

**Files:** (chỉ đọc, không sửa)

Chạy 1 lượt grep tổng để chắc mọi fix đã vào, không sót.

- [ ] **Step 1: Không còn subfolder wiki (A)**

Run: `grep -rn "wiki/decisions/\|wiki/patterns/\|wiki/concepts/\|wiki/troubleshooting/" skills/`
Expected: rỗng.

- [ ] **Step 2: Không còn hardcode path Gemini (G1)**

Run: `grep -rn "gemini" skills/`
Expected: rỗng.

- [ ] **Step 3: Thuật toán resolve slug có mặt (B)**

Run: `grep -n "ambiguous" skills/knowhow-lint/SKILL.md`
Expected: ≥1 hit.

- [ ] **Step 4: distill grep nội dung (C1)**

Run: `grep -n "grep -ril" skills/knowhow-distill/SKILL.md`
Expected: ≥1 hit.

- [ ] **Step 5: Forcing function (C2)**

Run: `grep -ni "≥ 5 item\|then-distill" skills/`
Expected: ≥2 hit.

- [ ] **Step 6: Provenance trỏ raw, không inbox (C3)**

Run: `grep -rn "source: inbox" skills/`
Expected: rỗng.

- [ ] **Step 7: status + confidence (C4)**

Run: `grep -c "status: active" skills/knowhow-capture/references/page-formats.md`
Expected: ≥6.

- [ ] **Step 8: Không còn "tự cải tiến" (D)**

Run: `grep -rn "tự cải tiến" specs/ skills/`
Expected: rỗng.

- [ ] **Step 9: rebuild-index + archive (F, H)**

Run: `grep -ni "rebuild index" skills/knowhow-lint/SKILL.md; grep -n "archive/" skills/knowhow-init/references/schema-template.md`
Expected: cả 2 có hit.

- [ ] **Step 10: idempotent + task.md (I, G2)**

Run: `grep -n "Idempotent" skills/knowhow-init/SKILL.md; ls specs/task.md`
Expected: cả 2 có hit.

- [ ] **Step 11: Commit cuối (nếu có thay đổi sót)**

```bash
cd /Users/tuanhv/Desktop/knowledge-engine
git add -A
git commit -m "chore: hoàn tất fix v1.1, verify toàn cục đối chiếu tiêu chí done" --allow-empty
```
