---
name: knowhow-lint
description: "Người dọn dẹp hệ thống knowhow: rà soát, dọn, và sửa cấu trúc cho kho .knowhow/. Dùng khi user muốn tìm và sửa vấn đề trong kho (link hỏng, page mồ côi, thiếu metadata, trùng wiki, lệch schema), sinh lại index/registry sau khi thêm nội dung, khôi phục page từ archive, hoặc đo kho có đang được dùng không. Trigger: 'knowhow lint', 'kiểm tra knowhow', 'rà soát kho', 'audit kho', 'dọn dẹp kho', 'rebuild index', gộp wiki (consolidation), kiểm tra mâu thuẫn, deep audit, metrics. KHÔNG dùng cho: linter code (eslint/pylint), review PR, audit file chung, hay thêm/sửa NỘI DUNG tri thức (đó là knowhow-capture/knowhow-distill)."
---

# Knowhow Lint

Rà soát sức khoẻ hệ thống knowhow. Sáu chế độ: **quick** (mặc định), **consolidation** (deep audit), **rebuild-index** (sinh lại file dẫn xuất), **schema-review** (tiến hoá cấu trúc), **restore** (khôi phục page/bó từ `archive/`), **metrics** (đo giá trị sử dụng).

Phân vai giữa hai trục đo: quick/consolidation đo sức khoẻ **cấu trúc** (kho có sạch không); metrics đo sức khoẻ **giá trị** (kho có đang được dùng không). Kho sạch mà không ai dùng vẫn là kho chết.

## Precondition

1. Xác định thư mục `.knowhow/` trong workspace hiện tại.
2. Kiểm tra `.knowhow/` tồn tại và chứa ít nhất 1 trong: `wiki/`, `skills/`, `workflows/`.
3. Nếu không tồn tại → thông báo user chưa có hệ thống knowhow, dừng.

## Chọn chế độ

- User không nói gì thêm, hoặc nói "quick", "lint" → chạy **Quick Lint**.
- User nói "consolidation", "deep", "audit" → chạy **Consolidation**.
- User nói "rebuild", "rebuild-index", "sinh lại index" → chạy **Rebuild Index**.
- User nói "schema-review", "review khuôn", "tiến hoá khuôn", "schema evolution" → chạy **Schema Review**.
- User nói "restore", "khôi phục", "lấy lại page", "phục hồi từ archive" → chạy **Restore**.
- User nói "metrics", "đo lường", "kho có đang được dùng không", "báo cáo sử dụng" → chạy **Metrics**.

---

## Chế độ 1: Quick Lint (mặc định)

Chạy nhanh, kiểm tra 5 hạng mục. Không đọc nội dung page, chỉ đọc metadata và link.

### 1a. Registry sync

- Đọc `wiki/index.md`: mỗi entry trỏ đến file → kiểm tra file tồn tại.
- File `.md` trong `wiki/` nhưng không có trong `wiki/index.md` → báo.
- Tương tự cho `skills/registry.md` và `workflows/registry.md`.

### 1b. Link integrity

- Tìm tất cả `[[...]]` trong body các page (wiki, skills, workflows).
- Trước khi resolve, **đọc tập type hợp lệ từ `SCHEMA.md`** (cột "Type" trong bảng Page Types) thay vì hardcode. Gọi tập này là `TYPES` (mặc định mới init: {decision, pattern, concept, troubleshooting, lesson}; sau tiến hoá có thể thêm, ví dụ `experiment`).
- File wiki có thể nằm **phẳng** (`wiki/<type>-<slug>.md`) hoặc **trong subfolder** (`wiki/<nhóm>/<type>-<slug>.md`). Resolve tìm ĐỆ QUY trong `wiki/`.
- Resolve mỗi link `[[X]]` theo thuật toán (match CHẶT type-slug, không glob đuôi tự do):
  1. Nếu `X` có dạng `<type>-<slug>` với `type ∈ TYPES` (và phần `<slug>` sau gạch KHÔNG rỗng): tìm đệ quy file khớp chính xác tên `<type>-<slug>.md` trong `wiki/`:
     ```bash
     find .knowhow/wiki -name "<type>-<slug>.md"
     ```
     Đúng 1 file → resolve OK. 0 → báo link hỏng. ≥2 (cùng tên ở 2 subfolder) → báo trùng vị trí.
  2. Nếu `X` là slug trần: với mỗi `type ∈ TYPES`, tìm đệ quy `find .knowhow/wiki -name "<type>-X.md"`. Đếm số TYPE có ít nhất 1 file khớp (KHÔNG phải tổng số file):
     - Đúng 1 type khớp và type đó có đúng 1 file → resolve OK.
     - 0 type khớp → báo link hỏng (ghi page nguồn + target).
     - 1 type khớp nhưng có ≥2 file (cùng type ở 2 subfolder) → báo **trùng vị trí** (không phải ambiguous).
     - ≥2 type khớp (cùng slug khác type) → báo **ambiguous**, yêu cầu link kèm type `[[<type>-X]]`.
  3. Skill/workflow: tìm đệ quy `find .knowhow/skills -name "X.md"` hoặc `find .knowhow/workflows -name "X.md"`.

### 1c. Frontmatter check

Mọi page phải có: `type`, `title`, `created`, `status`.

Thêm theo loại:
- **Skill**: `version`, `trigger`, `input`, `output`
- **Workflow**: `version`, `trigger`
- **Wiki (mọi type ∈ TYPES)**: `confidence`

Thiếu field nào → báo cụ thể file và field thiếu.

Field TUỲ CHỌN (KHÔNG báo thiếu): `promoted_from` và `reusable_across` ở skill; `promote_of` ở inbox item.

### 1d. Inbox backlog

- Liệt kê file trong `inbox/`.
- Tính tuổi item theo `captured_at` trong frontmatter (field thời gian chuẩn của inbox item, xem page-formats mục 1). Nếu thiếu `captured_at`, fallback mtime file. Cũ hơn 7 ngày → cảnh báo tồn đọng. (Lưu ý: inbox dùng `captured_at`, KHÔNG dùng `created` như wiki page.)

### 1e. Workflow-skill dependency

- Workflow reference `[[skill-name]]` → kiểm tra skill đó có trong `skills/registry.md`.
- Không có → báo.

### Output format (Quick Lint)

```markdown
## Quick Lint Report - YYYY-MM-DD

### Registry không đồng bộ (N items)
- [ ] Mô tả vấn đề

### Link hỏng (N items)
- [ ] Page X → [[Y]] nhưng Y không tồn tại

### Thiếu frontmatter (N items)
- [ ] File Z thiếu field: version

### Inbox tồn đọng (N items)
- [ ] inbox/abc.md (N ngày trước)

### Workflow dependency (N items)
- [ ] Workflow W reference [[S]] nhưng skill S không tồn tại

✅ Không có vấn đề nào (nếu clean)
```

Tạo báo cáo dạng artifact. Trình bày cho user duyệt.

### Fix hai bậc

**Bậc 1, tự fix ngay rồi báo trong report (KHÔNG chờ duyệt)**: chỉ áp cho sổ sách dẫn xuất, cùng bản chất với mode `rebuild-index` (vốn sinh lại toàn bộ registry không hỏi từng dòng):

- Registry thiếu entry mà file thật tồn tại → thêm entry vào index/registry.
- Registry thừa entry mà file thật không tồn tại → xoá entry.

Đánh dấu `✅ đã tự fix` trong report. Lý do được tự làm: index/registry là file dẫn xuất từ frontmatter, sửa chúng là đồng bộ sổ sách, không phải sửa tri thức.

**Bậc 2, trình user duyệt rồi mới fix (đụng nội dung hoặc phải phán đoán)**:

- Link hỏng → ưu tiên SỬA link (trỏ target đúng); chỉ GỠ link khi user xác nhận target không còn tồn tại. Khi gỡ, để lại slug dạng text (ví dụ ghi chú "(link cũ: old-slug, đã gỡ)") thay vì xoá trắng, để vết tích còn truy được mà không cần git.
- Frontmatter thiếu → bổ sung field với giá trị mặc định hợp lý.
- Inbox tồn đọng → nhắc user xử lý, không tự xoá.
- Workflow dependency → báo user review, không tự xoá reference.

---

## Chế độ 2: Consolidation (deep audit)

Đọc **NỘI DUNG** tất cả page trong `wiki/`, `skills/`, `workflows/`.

### Bước thực hiện

1. Đọc `references/consolidation-checklist.md` để nắm 8 hạng mục rà soát.
2. Chạy Quick Lint trước (gộp kết quả).
3. Đọc toàn bộ nội dung các page.
4. Rà soát theo 8 hạng mục trong checklist.
5. Tạo báo cáo consolidation.

### Output format (Consolidation)

```markdown
## Consolidation Report - YYYY-MM-DD

### Mâu thuẫn nội dung (N items)
- [ ] Page A nói X, Page B nói Y → đề xuất thống nhất

### Đề xuất gộp (N items)
- [ ] Page A + Page B có nội dung chồng chéo → gộp?

### Lỗi thời (N items)
- [ ] Page X không cập nhật từ DD/MM → review hoặc archive?
- [ ] Page Y `updated` cũ hơn 90 ngày, không có entry changelog mới → hạ `confidence` 1 bậc (high→medium→low)?

### Thiếu phủ (N items)
- [ ] Lĩnh vực Y chưa có knowhow

### Nhất quán thuật ngữ (N items)
- [ ] Khái niệm Z được gọi là A ở page P1, gọi là B ở page P2

### Workflow dependency outdated (N items)
- [ ] Workflow W dùng skill S version 1.0, nhưng S đã là version 2.0
```

Tạo báo cáo dạng artifact. Trình bày cho user duyệt.

### Sau khi user duyệt

Thực thi từng thay đổi đã được duyệt:
- Gộp page → tạo/cập nhật page đích, các page bị nuốt set `status: archived` và move vào `archive/` (KHÔNG xoá cứng), cập nhật registry và rewrite link `[[old-slug]]` thành `[[new-slug]]`.
- Thống nhất thuật ngữ → find-replace trên tất cả page liên quan.
- Archive page lỗi thời → chuyển vào `archive/`, set `status: archived`, xoá khỏi registry.
- Hạ confidence → sửa `confidence` trong frontmatter page xuống 1 bậc, ghi entry changelog.
- Bổ sung page thiếu phủ → tạo stub page, thêm vào registry.
- Cập nhật workflow dependency → sửa version reference.

---

## Chế độ 3: Rebuild Index (file dẫn xuất)

`wiki/index.md`, `skills/registry.md`, `workflows/registry.md` là **file dẫn xuất** từ frontmatter các page. Khi merge conflict ở 3 file này, không cần giải tay, sinh lại.

### Bước thực hiện

1. Quét frontmatter mọi file trong `wiki/` ĐỆ QUY (gồm subfolder, trừ index.md, log.md): đọc `type`, `title`, `tags`, `updated`. Dùng `find .knowhow/wiki -name "*.md" -not -name "index.md" -not -name "log.md"`.
2. Sinh `wiki/index.md`, group theo `type`. Tập type ĐỌC TỪ bảng Page Types trong `SCHEMA.md` (không cố định 4 heading. Nếu schema đã thêm type mới như `experiment`, sinh thêm heading tương ứng). Mỗi dòng `- [[<type>-<slug>]] - <title>`. Heading mỗi type là giá trị type viết HOA chữ cái đầu (decision → `## Decision`, experiment → `## Experiment`), nhất quán với seed của `knowhow-init`.
3. Quét frontmatter mọi file trong `skills/` (đệ quy, `find .knowhow/skills -name "*.md" -not -name "registry.md"`) (trừ registry.md): đọc `title`, `trigger`, `version`, `tags`, `updated`. Sinh `skills/registry.md` theo format page-formats mục 6.1 (cột `Khi nào dùng` lấy từ `trigger`; skill cũ thiếu `trigger` → để ô trống và sẽ bị frontmatter check 1c báo; bỏ qua `promoted_from`/`reusable_across` vì registry không có cột tương ứng), sort alphabet.
4. Quét frontmatter mọi file trong `workflows/` (đệ quy, `find .knowhow/workflows -name "*.md" -not -name "registry.md"`) (trừ registry.md): đọc `title`, `trigger`, `skills_used`, `version`, `tags`, `updated`. Sinh `workflows/registry.md` theo format mục 6.2 (cột `Khi nào dùng` lấy từ `trigger`, cột `Tags` lấy từ `tags`; workflow cũ thiếu `trigger` → để ô trống và sẽ bị frontmatter check 1c báo), sort alphabet.
5. Ghi log: `## [YYYY-MM-DD] lint | Rebuild index + 2 registry từ frontmatter`.

---

## Chế độ 4: Schema Review (tiến hoá cấu trúc)

Tổng hợp tín hiệu "khuôn không vừa" → đề xuất diff lên `SCHEMA.md` → user duyệt → migrate. Đây là vòng lặp tiến hoá *bên trên* cơ chế cải tiến nội dung.

**Reference bắt buộc đọc trước**: `references/schema-review.md` (ngưỡng + migration playbook + bước chung cuối batch).

### Precondition

- `.knowhow/schema-signals.md` PHẢI tồn tại (init tạo). Nếu thiếu: tạo lại file rỗng tối thiểu gồm dòng tiêu đề `# Schema Signals`, rồi hai heading `## Đang chờ xử lý` và `## Đã xử lý` (giống template init), rồi tiếp tục, coi như chưa có tín hiệu sự kiện nào. KHÔNG dừng. (Lưu ý: `schema-signals.md` là file meta top-level, KHÔNG phải page nên không chịu frontmatter check 1c.)

### Flow

1. **Đọc sổ tín hiệu**: đọc `.knowhow/schema-signals.md`, lấy các dòng trong mục "Đang chờ xử lý" (BỎ QUA mục "Đã xử lý"). BỎ QUA các dòng loại `promote-candidate` (đó là việc của `knowhow-distill`, không phải schema-review).
2. **Quét sống**: tính tín hiệu trạng thái tại thời điểm chạy (đếm page mỗi type, cụm tag, file/folder phình, orphan). Lệnh cụ thể trong `references/schema-review.md` mục 2. KHÔNG ghi tín hiệu trạng thái vào sổ.
3. **Áp ngưỡng**: đối chiếu tín hiệu (sự kiện + trạng thái) với 4 ngưỡng trong reference mục 3. Chỉ những gì VƯỢT ngưỡng mới thành đề xuất. Dưới ngưỡng → giữ trong sổ, không báo.
4. **Sinh đề xuất diff**, nhóm theo 4 loại thay đổi. Trình bày cho user (xem Output format dưới). Nếu KHÔNG có tín hiệu nào vượt ngưỡng → hiển thị `✅ Không đủ ngưỡng nào` và DỪNG, không sang bước 5-6.
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
- **Reversibility (kép)**: gốc không cần git, migrate không xoá cứng file nào; page bị rút/đổi đi qua `archive/` + `status: archived/deprecated` nên khôi phục được bằng cách move ngược + đổi status. Nếu không gian làm việc có git, `git revert` là lớp khôi phục cộng thêm cho cả batch.
- **Không auto-migrate**: mọi batch gate qua user.

---

## Chế độ 5: Restore (khôi phục từ archive)

Hiện thực thao tác của lớp gốc trong "reversibility kép": lấy lại page/bó/inbox item đã bị rút khỏi hệ thống (move vào `archive/` + `status: archived/deprecated`) mà KHÔNG cần git. Đây là phần đối xứng của các thao tác phá huỷ ở distill (GỘP/deprecate) và consolidation/schema-review (archive).

> Nếu kho có git và muốn hoàn tác cả một batch migrate gần nhất, `git revert` sạch hơn. `restore` dành cho khôi phục LẺ một vài file, hoặc kho không dùng git.

### Bước thực hiện

1. **Xác định mục tiêu**:
   - Input là slug/tên file cụ thể → tìm trong `archive/` (đệ quy): `find .knowhow/archive -name "*<slug>*"`.
   - Input rỗng → liệt kê nội dung `archive/` (gồm `archive/inbox/`) cho user chọn, kèm `status` và ngày archive nếu đọc được.
2. **Suy ra đích khôi phục** từ loại file:
   - Wiki page (tên có prefix `<type>-`) → `wiki/` (hoặc subfolder `wiki/<type>/` nếu type đã tách).
   - Skill → `skills/`. Workflow → `workflows/`.
   - Inbox item (nằm trong `archive/inbox/`) → `inbox/`.
3. **Move file** từ `archive/` về đích. KHÔNG ghi đè nếu đích đã có file trùng tên: dừng, báo user xung đột để xử lý tay.
4. **Đổi `status`** trong frontmatter về `active` (page/bó). Inbox item không có `status`, bỏ qua bước này.
5. **Kiểm link**: nếu page bị khôi phục từng bị GỘP (link `[[old-slug]]` đã bị rewrite sang `[[new-slug]]` ở nơi khác), rewrite ngược KHÔNG tự động (rủi ro). Chỉ cảnh báo user rà link thủ công, và đảm bảo bản thân page khôi phục không chứa link hỏng (chạy resolve mục 1b cho riêng nó).
6. **Rebuild index**: chạy mode `rebuild-index` để page/bó hiện lại trong index/registry.
7. **Log**: `## [YYYY-MM-DD] lint | Restore <đường-dẫn-đích> từ archive`.

### Nguyên tắc

- Restore là thao tác có chủ đích của user, gate qua xác nhận từng file. KHÔNG khôi phục hàng loạt tự động.
- KHÔNG xoá bản trong `archive/` sau khi restore trừ khi user yêu cầu (giữ để vẫn trace được lịch sử).

---

## Chế độ 6: Metrics (đo giá trị sử dụng)

Trả lời một câu duy nhất: **kho này có đang sinh giá trị không, hay chỉ phình ra?** Toàn bộ tính từ `wiki/log.md` + frontmatter + `schema-signals.md`, không cần công cụ ngoài. Chế độ này CHỈ báo cáo và đề xuất, không sửa gì.

### Nguồn dữ liệu

```bash
# Toàn bộ entry log, parse được theo op
grep "^## \[" .knowhow/wiki/log.md
# Entry usage của run (knowhow-run ghi 1 dòng mỗi lần chạy bó)
grep "^## \[" .knowhow/wiki/log.md | grep "| run |"
```

### Chỉ số (tính cho 90 ngày gần nhất trừ khi ghi khác)

| # | Chỉ số | Cách tính | Ngưỡng đáng lo |
|---|---|---|---|
| 1 | Inbox tồn | số item + tuổi trung vị theo `captured_at` | trung vị > 7 ngày |
| 2 | Nhịp tích luỹ | số entry `capture` và `distill` trong 30 ngày | 0 capture trong 30 ngày |
| 3 | Query hiệu quả | tổng entry `query`; tỉ lệ dòng có `emit query-miss: có` | miss > 1/3 |
| 4 | Reuse mỗi bó | đếm entry `run` group theo slug bó; tỉ lệ `vướng:` mỗi bó | bó 0 lần run trong 2 quý; bó có ≥2 lần `vướng` |
| 5 | Độ tươi | % page `status: active` có `updated` < 90 ngày | < 60% |
| 6 | Lỗi lặp | troubleshooting page có ≥ 2 entry Changelog ghi nhận tái diễn trong 90 ngày | bất kỳ page nào dính |
| 7 | Hành động treo | lesson page có mục "Hành động hệ thống" còn ghi "chờ distill" | treo > 30 ngày |
| 8 | Promote chảy | dòng `promote-candidate` đang chờ vs đã xử lý trong `schema-signals.md` | chờ ≥ 3 phiếu cùng slug mà chưa distill |

### Kiểm tra định tính (quan trọng hơn mọi con số)

Lấy 3 entry `run | <bó> | ok` gần nhất, nêu lại thành câu: "3 lần gần nhất kho đỡ việc thật: ...". Nếu KHÔNG tìm nổi 3 entry trong quý, ghi cảnh báo đậm: kho đang thành nghĩa địa tri thức, vấn đề không nằm ở nội dung mà ở thói quen tiêu thụ (xem lại agent config có nhắc tra registry không).

### Output format (Metrics)

```markdown
## Metrics Report - YYYY-MM-DD (cửa sổ: 90 ngày)

### Dòng chảy: capture N → distill M → inbox tồn K (tuổi trung vị X ngày)
### Tiêu thụ: query Q lần (miss R%), run S lần trên T bó
### Bằng chứng giá trị: [3 lần run ok gần nhất, hoặc cảnh báo nghĩa địa]

### Đề xuất hành động (N items)
- [ ] Bó [[X]] 2 quý không ai dùng → xét deprecated?
- [ ] Bó [[Y]] vướng 3 lần cùng bước 4 → chạy distill refine?
- [ ] Lesson [[Z]] hành động hệ thống treo 45 ngày → chạy distill?
- [ ] Query-miss cụm tag:<chủ-đề> lặp → đã có trong sổ tín hiệu, chạy schema-review?
```

Trình bày cho user. Mọi hành động user duyệt thì chuyển cho skill tương ứng (distill, consolidation), metrics KHÔNG tự thực thi.

---

## Log

Ghi vào `wiki/log.md`:

```
## [YYYY-MM-DD] lint | Quick lint: N vấn đề, M đã fix
## [YYYY-MM-DD] lint | Consolidation: N vấn đề, M đã fix
## [YYYY-MM-DD] lint | Schema-review: N đề xuất, M migrate
## [YYYY-MM-DD] lint | Restore wiki/<type>-<slug>.md từ archive
## [YYYY-MM-DD] lint | Metrics: capture N, distill M, query Q (miss R%), run S
```
