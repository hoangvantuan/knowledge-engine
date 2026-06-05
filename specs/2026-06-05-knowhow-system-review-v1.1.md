# Knowhow System: Review & Fix Spec v1.1

**Ngày**: 2026-06-05
**Quan hệ**: Bổ sung cho [2026-06-05-knowhow-system-design.md](2026-06-05-knowhow-system-design.md) v1.0. File này KHÔNG thay thế spec gốc, chỉ chốt các quyết định sửa sau review.
**Trạng thái**: 4 skills đã implement (xem [task.md](task.md)). Đây là danh sách fix trước khi dùng thật.

## La bàn

Review code thật phát hiện: 2 bug đường dẫn (3 file quy ước wiki khác nhau + 1 hardcode path Gemini trong distill), 1 lỗ hổng logic (distill mù nội dung), 1 root cause (distill là điểm chết duy nhất), và vài chỗ underspec. Spec này chốt cách sửa, theo thứ tự ưu tiên. Bản này đã gom thêm phát hiện review (G1 path Gemini, G2 task.md thiếu) và chốt 5 chỗ underspec (A1-A5).

## Lõi vấn đề (đọc trước)

Giá trị hệ thống = **chất lượng × tần suất** của bước `distill`. Bỏ distill thì raw + inbox chỉ là đống ghi chú rời. Nhưng distill hiện vừa **mù nội dung** (chỉ đọc registry titles) vừa **không có lực đẩy chạy** (không trigger). Mọi fix bên dưới phục vụ làm bước distill đáng tin và chạy đều.

---

## Thứ tự ưu tiên

| Hạng | Fix | Lý do làm trước |
|------|-----|-----------------|
| P0 | A. Chốt 1 quy ước đường dẫn | Bug thật, mọi thứ khác phụ thuộc |
| P0 | B. Thuật toán resolve `[[slug]]` | Hệ quả của A, lint sai nếu không có |
| P1 | C. Gia cố distill (grep + forcing function) | Lõi, quyết định giá trị hệ thống |
| P1 | D. Đổi mô tả cho trung thực | Chánh Ngữ, sửa kỳ vọng |
| P2 | E-H. Vá nhỏ | Bug biên, không chặn vận hành |

---

## P0-A: Chốt một quy ước đường dẫn wiki

**Vấn đề**: 3 nguồn nói 3 kiểu.
- [knowhow-init/SKILL.md:44](skills/knowhow-init/SKILL.md#L44) tạo subfolder `wiki/decisions/`, `wiki/patterns/`...
- [page-formats.md:73](skills/knowhow-capture/references/page-formats.md#L73) ghi phẳng `wiki/decision-slug.md`.
- [schema-template.md:41](skills/knowhow-init/references/schema-template.md#L41) naming dùng tên không prefix `rest-to-graphql.md`.

Kết quả: subfolder rỗng vĩnh viễn, file ghi sai chỗ.

**Quyết định**: Dùng **wiki phẳng + prefix type trong tên file**.
- Đường dẫn: `wiki/<type>-<slug>.md`. Ví dụ: `wiki/decision-rest-to-graphql.md`, `wiki/pattern-retry-with-jitter.md`.
- Lý do chọn phẳng: dễ cho grep ở fix C, ít thư mục con để agent điều hướng, tên file tự mang type.

**Sửa file**:
1. `knowhow-init/SKILL.md` Bước 2: bỏ tạo 4 subfolder `decisions/ patterns/ concepts/ troubleshooting/`. Chỉ tạo `wiki/` phẳng (vẫn giữ `index.md`, `log.md`).
2. `knowhow-init/references/schema-template.md` bảng Page Types: đổi cột thư mục thành `wiki/` cho cả 4 type. Naming Conventions: ghi rõ `wiki/<type>-<slug>.md`.
3. `knowhow-capture/references/page-formats.md`: giữ nguyên (đã đúng), chỉ đảm bảo header mỗi mục ghi `wiki/<type>-slug.md` nhất quán.

### P0-A2: Chốt cách reference file `page-formats.md` (gom từ review G1)

**Vấn đề**: Cùng họ bug đường dẫn. [distill:78](skills/knowhow-distill/SKILL.md#L78) hardcode đường dẫn tuyệt đối gắn Gemini: `~/.gemini/config/skills/knowhow-capture/references/page-formats.md`. Nhưng [capture:85,112](skills/knowhow-capture/SKILL.md#L85) reference bằng đường dẫn tương đối `references/page-formats.md`. Deploy skill ở chỗ khác (ví dụ `~/.claude/skills/`) là distill gãy reference.

**Quyết định**: distill KHÔNG hardcode đường dẫn tuyệt đối. Vì `page-formats.md` nằm trong skill `knowhow-capture` (skill khác), distill tham chiếu bằng **đường dẫn tương đối từ skill capture**: `../knowhow-capture/references/page-formats.md`. Nếu không resolve được, distill tự fallback về format frontmatter cơ bản (logic fallback đã có sẵn ở distill Bước 5.1).

**Sửa file**: `knowhow-distill/SKILL.md` Bước 5.1: đổi `~/.gemini/config/skills/knowhow-capture/references/page-formats.md` thành `../knowhow-capture/references/page-formats.md`.

## P0-B: Thuật toán resolve `[[slug]]`

**Vấn đề**: [lint 1b](skills/knowhow-lint/SKILL.md#L34) "resolve link thành file path" nhưng không định nghĩa. index.md viết `[[rest-to-graphql]]`, file thật `decision-rest-to-graphql.md`. Không khớp → lint báo link hỏng giả.

**Quyết định**: Định nghĩa slug và resolve rõ ràng. Match **chặt theo type-slug**, không glob đuôi tự do (chống match nhầm hậu tố, ví dụ `[[graphql]]` không được match `decision-rest-to-graphql.md`).

Thuật toán resolve `[[X]]`:
1. **Nếu `X` có dạng `<type>-<slug>`** (type ∈ {decision, pattern, concept, troubleshooting}): khớp chính xác file `wiki/<type>-<slug>.md`. Đây là dạng tường minh, không ambiguous.
2. **Nếu `X` là slug trần** (không prefix type): thử khớp chính xác từng ứng viên `wiki/decision-X.md`, `wiki/pattern-X.md`, `wiki/concept-X.md`, `wiki/troubleshooting-X.md`. Đếm số file tồn tại.
   - Đúng 1 file → resolve thành công.
   - 0 file → báo link hỏng (ghi page nguồn + target).
   - ≥2 file (cùng slug khác type) → báo ambiguous, yêu cầu link kèm type `[[decision-X]]`.
3. **Skill/workflow**: khớp chính xác `skills/X.md` hoặc `workflows/X.md` (không prefix type, slug là tên file).

`slug` = phần sau prefix type, không gồm type. Ví dụ file `decision-rest-to-graphql.md` có slug `rest-to-graphql`.

**Sửa file**: `knowhow-lint/SKILL.md` mục 1b: thêm thuật toán resolve trên. `schema-template.md` mục Cross-referencing: ghi rõ cú pháp slug.

## P1-C: Gia cố distill (lõi)

### C1. Grep nội dung trước khi quyết định (chống mù)

**Vấn đề**: [distill Bước 3.2](skills/knowhow-distill/SKILL.md#L52) bảo "so sánh nội dung với registry", nhưng registry chỉ có title + mô tả 1 dòng. So sánh nội dung là bất khả thi.

**Quyết định**: Chèn bước grep nội dung thật.
```
Bước 3.2 (mới): Với mỗi inbox item, rút 3-5 từ khoá + tags rồi chạy:
   grep -ril "<từ khoá>" .knowhow/wiki .knowhow/skills .knowhow/workflows
   Đọc các file hit. CHỈ sau đó mới áp bảng quyết định tạo mới / cập nhật.
```
- `tags` thành khoá nối bắt buộc: capture phải gán tag nhất quán để grep theo tag hiệu quả.
- Lưới an toàn: `knowhow-lint consolidation` chạy định kỳ bắt trùng mà grep lọt.

### C2. Forcing function (chống không chạy)

**Vấn đề**: Không gì đảm bảo distill chạy. Capture xong, inbox đầy, user quên. Compounding sụp.

**Quyết định**: Thêm lực đẩy.
- `agent-config-snippet.md`: thêm dòng "Khi `.knowhow/inbox/` có ≥ 5 item hoặc item cũ > 7 ngày, chủ động nhắc user chạy `knowhow-distill`."
- Cho user solo: cho phép gọi capture với cờ `--then-distill` chạy liền 2 bước trong 1 phiên duyệt.

**Sửa file**: `knowhow-distill/SKILL.md` (C1), `knowhow-init/references/agent-config-snippet.md` (C2).

### C3. Provenance sống sót sau distill

**Vấn đề**: [capture:88](skills/knowhow-capture/SKILL.md#L88) chỉ lưu raw khi có file ngoài. Hội thoại không vào raw. [distill:107](skills/knowhow-distill/SKILL.md#L107) xoá inbox. Changelog trỏ `inbox/...` đã bị xoá → trace chết.

**Quyết định**:
- Capture **luôn** lưu trích đoạn nguồn (cả hội thoại) vào `raw/YYYY-MM-DD-slug.md`.
- Tên field provenance: **dùng `source_file` đã có sẵn** trong inbox frontmatter (xem [page-formats.md:21](skills/knowhow-capture/references/page-formats.md#L21)), KHÔNG đẻ field `source:` mới. Lý do: tránh 3 tên cho 1 ý (`source_file`, `captured_from`, `source`). `source_file` từ "để trống nếu không có file ngoài" đổi thành **bắt buộc trỏ `raw/...`** cho mọi item (gồm hội thoại).
- Distill changelog ghi text `(source: raw/...)`, lấy từ `source_file` của inbox item, không phải `inbox/...`. (Lưu ý: `source` trong changelog là chữ mô tả, không phải tên field frontmatter.)

**Sửa file**: `knowhow-capture/SKILL.md` Bước 4, `knowhow-capture/references/page-formats.md` (mô tả lại `source_file` thành bắt buộc trỏ raw/), `knowhow-distill/SKILL.md` Bước 5 (changelog + log).

### C4. Metadata sống

**Vấn đề**: `confidence` không có trong schema-template và không có vòng đời. `concept` thiếu hẳn field. distill nói "đánh dấu deprecated" nhưng không có field `status`.

**Quyết định**:
- Thêm `status: active | deprecated | archived` vào frontmatter **mọi page, gồm cả skill và workflow** (không chỉ wiki). Distill set field này khi deprecate, thay vì "đánh dấu" mơ hồ. Mặc định khi tạo: `active`. lint 1c (frontmatter-check) thêm `status` vào danh sách field bắt buộc của mọi page.
- Thêm `confidence` vào cả `concept` (chỉ áp cho 4 wiki type; skill/workflow dùng `version`, không dùng `confidence`).
- **Vòng đời `confidence` đếm theo số entry trong phần Changelog của page** (không cần cơ chế đếm riêng):
  - capture set `low` (item mới, 1 lần ghi nhận).
  - distill nâng khi page được cập nhật lặp lại (grep C1 trỏ về page cũ → CẬP NHẬT → thêm 1 entry changelog): **≥2 entry → `medium`, ≥3 entry → `high`**.
  - lint (consolidation) **hạ** confidence 1 bậc khi page có `updated` cũ hơn 90 ngày và không có entry changelog mới trong khoảng đó.

**Sửa file**: `schema-template.md` (định nghĩa lifecycle `status` + `confidence`), `page-formats.md` (thêm `confidence` vào concept; thêm `status` vào frontmatter mọi type gồm skill/workflow), `knowhow-distill/SKILL.md` (set/nâng confidence + status theo Changelog), `knowhow-lint/SKILL.md` 1c (bắt `status`) + consolidation (hạ confidence).

## P1-D: Đổi mô tả cho trung thực

**Vấn đề**: "bộ não tự cải tiến" hứa mức tự động mà thiết kế cố tình không có (mọi bước gate qua user duyệt). Đây là human-in-loop curated accumulation.

**Quyết định**: Đổi mô tả trong spec gốc + init SKILL thành: "nghi thức tích luỹ tri thức có cấu trúc, AI viết và người duyệt". Giữ tinh thần Karpathy, bỏ chữ "tự".

**Sửa file**: grep toàn bộ cụm "tự cải tiến" trong repo, sửa MỌI chỗ (không chỉ dòng mục tiêu):
- `design.md`: dòng mục tiêu (dòng 5), heading "Vòng đời tự cải tiến" (dòng 442).
- `knowhow-init/SKILL.md`: Tổng quan (dòng 10).
- `knowhow-distill/SKILL.md`: "Hệ thống knowhow tự cải tiến bằng cách tích lũy" (dòng 10).

---

## P2: Vá nhỏ

| # | Vấn đề | File | Cách sửa |
|---|--------|------|----------|
| E | Đa agent oversell: skills vận hành chỉ chạy trên Gemini/Antigravity, agent khác chỉ đọc | `schema-template.md` onboarding | Thêm dòng: "Agent ngoài Gemini/Antigravity chỉ ĐỌC knowhow. Capture/distill/lint chạy trên Antigravity." |
| F | `index.md` + `registry.md` là điểm tranh chấp merge | `knowhow-lint/SKILL.md` | Coi 3 file `wiki/index.md`, `skills/registry.md`, `workflows/registry.md` là file dẫn xuất. Thêm chế độ `rebuild-index` cho lint: quét frontmatter (`type`, `title`, `tags`, `version`, `updated`) mọi file trong `wiki/`, `skills/`, `workflows/`, sinh lại cả 3 file. index.md group theo `type` (4 heading). registry sinh theo format ở page-formats mục 6. Conflict ở 3 file này thành rác vứt được (rebuild lại). |
| G | GỘP/deprecate không rewrite inbound link | `knowhow-distill/SKILL.md` Bước 5.3 | Khi gộp/deprecate, grep `[[old-slug]]` toàn repo, sửa hoặc xoá link nguồn. |
| H | `archive/` lint dùng nhưng init không tạo, SCHEMA không nhắc | `knowhow-init/SKILL.md`, `schema-template.md` | Init tạo `archive/`. SCHEMA mô tả mục đích. |
| I | `init` Bước 5 append snippet không idempotent → chạy lại nhân đôi | `knowhow-init/SKILL.md` Bước 5 | Trước khi append, grep `## Knowhow` trong file config. Nếu có rồi → bỏ qua. |

---

## Tiêu chí done cho v1.1

- [ ] Một quy ước đường dẫn duy nhất, 4 file đồng bộ (A)
- [ ] Thuật toán resolve slug rõ, lint không báo hỏng giả (B)
- [ ] Distill grep nội dung trước khi quyết định (C1)
- [ ] Có forcing function nhắc distill (C2)
- [ ] Trace sống sót sau khi inbox dọn (C3)
- [ ] `status` + `confidence` có field và vòng đời rõ (C4)
- [ ] Mô tả bỏ chữ "tự cải tiến" mọi chỗ (D)
- [ ] 5 vá nhỏ E-I xong
- [ ] Bug path Gemini trong distill đã sửa thành đường dẫn tương đối (G1 / P0-A2)
- [ ] `task.md` tồn tại, mô tả kịch bản test end-to-end (G2)
- [ ] Test end-to-end ([task.md](task.md)) chạy lại sau khi sửa
