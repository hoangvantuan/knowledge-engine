---
name: knowhow-distill
description: "Đúc kết knowhow từ inbox thành wiki pages, skills, workflows chính thức. Đọc cái đã có trước, ưu tiên cập nhật/cải tiến hơn tạo mới. AI đề xuất, user duyệt từng item. Trigger: 'knowhow distill', 'đúc kết knowhow', 'xử lý inbox', khi inbox có nội dung chờ, hoặc user muốn chuyển ghi nhận thành tri thức chính thức."
---

# Knowhow Distill

> **QUY TẮC SỐ 1: LUÔN đọc `wiki/index.md` + `skills/registry.md` + `workflows/registry.md` TRƯỚC KHI xử lý inbox. Ưu tiên cải tiến cái cũ hơn tạo mới.**
>
> Tại sao? Hệ thống knowhow tích luỹ giá trị bằng cách gộp tri thức, không phân mảnh. Tạo mới khi đã có page tương tự sẽ phân mảnh tri thức, khiến tìm kiếm khó hơn và kiến thức mâu thuẫn nhau. Cập nhật page cũ giữ tri thức tập trung, dễ tra cứu, ngày càng sâu.

## Precondition

1. Kiểm tra `.knowhow/` tồn tại trong project root.
2. Kiểm tra `.knowhow/inbox/` có ít nhất 1 file.
3. Nếu inbox rỗng → báo "Không có gì để đúc kết." rồi dừng.

## Bảng quyết định hành động

| Tình huống | Hành động | Ví dụ |
|---|---|---|
| Chưa có gì liên quan | **TẠO MỚI** page | Lần đầu gặp pattern retry |
| Đã có page, knowhow bổ sung | **CẬP NHẬT** page cũ, thêm nội dung, ghi changelog | Thêm edge case vào retry.md |
| Đã có page, knowhow thay thế | **SỬA** page cũ, set status: deprecated cho cách cũ | Đổi từ exponential sang jitter |
| Đã có skill/workflow, thiếu/thừa bước | **REFINE** skill/workflow, tăng version | Skill deploy thiếu bước verify |
| Nhiều page nhỏ cùng chủ đề | **GỘP** thành 1 page chất lượng hơn | 3 page error handling → 1 |
| Không đáng lưu | **BỎ QUA**: move inbox item vào `archive/inbox/` (không xoá cứng) | Thông tin quá cụ thể, không tái sử dụng |

## Flow distill

### Bước 1: Đọc registries

Đọc 3 file registry để biết tri thức đã có:

- `wiki/index.md` → danh sách wiki pages đã có (title, tags, mô tả ngắn)
- `skills/registry.md` → danh sách skills đã có (tên, version, mô tả)
- `workflows/registry.md` → danh sách workflows đã có (tên, version, mô tả)

Ghi nhận toàn bộ vào context. Đây là bước quan trọng nhất: không đọc registry → không biết cái đã có → sẽ tạo trùng.

### Bước 2: Đọc inbox

- Liệt kê tất cả file trong `.knowhow/inbox/`.
- Đọc nội dung từng file.
- Ghi nhận metadata: nguồn gốc, ngày tạo, tags (nếu có).

### Bước 3: Phân tích và đề xuất

Với mỗi inbox item:

1. **Phân loại**: wiki page (decision/pattern/concept/troubleshooting) hay skill hay workflow? (xem quy tắc phân loại bên dưới)
2. **Tìm trùng (grep nội dung thật, chống mù)**: Registry chỉ có title + mô tả 1 dòng, KHÔNG đủ để so sánh nội dung. Với mỗi inbox item:
   - Rút 3-5 từ khoá từ tiêu đề + `tags` của item.
   - Chạy grep tìm page liên quan:
     ```bash
     grep -ril "<từ khoá>" .knowhow/wiki .knowhow/skills .knowhow/workflows
     ```
   - Đọc các file hit. CHỈ sau khi đọc nội dung thật mới áp bảng quyết định (tạo mới / cập nhật / sửa / refine / gộp / bỏ qua).
   - Lưới an toàn: `knowhow-lint consolidation` chạy định kỳ bắt trùng mà grep lọt.

> **Lưu ý**: Capture phải gán `tags` nhất quán để grep theo tag hiệu quả. Item không tag → grep chỉ dựa từ khoá tiêu đề, dễ lọt trùng.

3. **Quyết định hành động**: Dùng bảng quyết định ở trên.

Trình bày đề xuất cho user. Mỗi item gồm:

```
📥 inbox/[tên-file].md
   → Hành động: TẠO MỚI / CẬP NHẬT / SỬA / REFINE / GỘP / BỎ QUA
   → Page đích: [type]/[slug].md (nếu cập nhật/sửa/refine/gộp)
   → Preview: [mô tả ngắn 1-2 câu nội dung sẽ thay đổi]
```

### Bước 3.5: Phát tín hiệu strain (tiến hoá cấu trúc)

Trong lúc phân loại + đề xuất, để ý hai dấu hiệu "khuôn không vừa". Nếu gặp, GHI tín hiệu vào `.knowhow/schema-signals.md` (mục "Đang chờ xử lý"). KHÔNG tự đổi khuôn, đó là việc của `knowhow-lint schema-review`. Giả định init đã tạo `.knowhow/schema-signals.md` với sẵn heading; nếu thiếu, tạo lại theo template của init trước khi ghi.

**Khi nào emit `no-fit-type`**: item rõ ràng là một *loại tri thức* không nằm trong 4 wiki type {decision, pattern, concept, troubleshooting} nhưng buộc phải xếp tạm vào wiki type gần nhất. Ví dụ: "kết quả thí nghiệm", "nguồn tham khảo cần lưu", "runbook vận hành". Dấu hiệu: bạn thấy mình miễn cưỡng chọn type vì không cái nào khớp.

**Khi nào emit `adhoc-section`**: khi tạo/cập nhật page, bạn phải thêm một section KHÔNG có trong template chuẩn của type đó (xem `../knowhow-capture/references/page-formats.md`). Kiểm tra section này có lặp ở page khác cùng type không:
```bash
grep -rl '## <tên section>' .knowhow/wiki
```
Nếu section đã xuất hiện ở ≥ 1 page khác cùng type → emit `adhoc-section`. Nếu chưa thấy ở đâu, CHƯA cần emit, chờ nó lặp lại.

**Cách ghi**: chèn dòng tín hiệu NGAY DƯỚI heading `## Đang chờ xử lý` trong `.knowhow/schema-signals.md`. KHÔNG append cuối file (cuối file là vùng `## Đã xử lý`). Chèn bằng awk:
```bash
awk '/^## Đang chờ xử lý$/{print; print "- [YYYY-MM-DD] distill | no-fit-type | <chi tiết ngắn> | related: tag:<chủ-đề>"; next} 1' \
  .knowhow/schema-signals.md > /tmp/ss && mv /tmp/ss .knowhow/schema-signals.md
```
Dòng tín hiệu theo đúng format `- [YYYY-MM-DD] distill | <loại> | <chi tiết ngắn> | related: tag:<chủ-đề>`, với `<loại>` ∈ no-fit-type | adhoc-section.

Ví dụ:
```
- [2026-06-05] distill | no-fit-type | item về kết quả thí nghiệm, không vừa decision/pattern/concept/troubleshooting | related: tag:experiment
- [2026-06-05] distill | adhoc-section | section "Metrics đo được" ở page experiment-ab-test | related: tag:experiment
```

> **Quan trọng**: emit tín hiệu KHÔNG chặn distill. Vẫn xếp item vào wiki page tốt nhất hiện có và xử lý bình thường. Tín hiệu chỉ là ghi chú cho lần `schema-review` sau.

### Bước 4: User duyệt

Hỏi user duyệt từng item: đồng ý / sửa / bác bỏ.

Cho phép user thay đổi:
- Hành động (ví dụ: đổi từ TẠO MỚI sang CẬP NHẬT)
- Page đích
- Loại (wiki/skill/workflow)

### Bước 5: Thực thi

Với mỗi item được duyệt, thực hiện theo thứ tự:

1. **Tạo hoặc cập nhật page** theo format chuẩn.
   - Format reference: `../knowhow-capture/references/page-formats.md` (đường dẫn tương đối từ thư mục skill này). Nếu không resolve được, dùng format frontmatter cơ bản bên dưới.
   - Format frontmatter cơ bản: title, type, tags, created, updated.
   - **Skill: BẮT BUỘC điền `trigger`** trong frontmatter (mô tả "khi nào nên dùng skill này"). Đây là tín hiệu để `knowhow-run` match task sang skill; thiếu nó registry sẽ trống cột `Khi nào dùng`. Khi REFINE skill cũ chưa có `trigger`, bổ sung luôn trong lần sửa này.

2. **Cập nhật registry tương ứng**:
   - Tạo/xoá wiki page → cập nhật `wiki/index.md`
   - Tạo/cập nhật skill → cập nhật `skills/registry.md`
   - Tạo/cập nhật workflow → cập nhật `workflows/registry.md`

3. **Cross-referencing**:
   - Thêm `related: [...]` trong frontmatter nếu page mới liên quan page cũ.
   - Thêm `[[...]]` link trong body khi mention page khác.
   - Cập nhật page cũ thêm reference ngược đến page mới (nếu cần).

3b. **Rewrite inbound link khi GỘP/deprecate**: Khi gộp page hoặc set deprecated, page khác có thể đang trỏ `[[old-slug]]`. Grep toàn repo và sửa:
   ```bash
   grep -rln "\[\[old-slug\]\]" .knowhow
   ```
   - GỘP: đổi `[[old-slug]]` → `[[new-slug]]` ở mọi file nguồn.
   - Deprecate (vẫn giữ page): để link nguyên nhưng đảm bảo page đích có `status: deprecated` để người đọc biết.

4. **Changelog**: Ghi dòng changelog cuối page bị thay đổi. Lấy nguồn từ `source_file` của inbox item (trỏ `raw/...`), KHÔNG trỏ `inbox/...` (inbox sẽ được dọn sang `archive/inbox/` ở mục 7, đường dẫn `inbox/...` sẽ không còn ổn định).
   ```
   ## Changelog
   - YYYY-MM-DD: [mô tả thay đổi] (source: raw/YYYY-MM-DD-slug.md)
   ```

5. **Version**: Tăng version trong frontmatter nếu skill/workflow bị sửa.

5b. **Lifecycle metadata**:
   - `status`: page mới set `active`. Khi hành động là SỬA (thay cách cũ), set page/section cũ `status: deprecated`. Khi GỘP, page bị nuốt set `status: archived`.
   - `confidence` (chỉ wiki, không áp skill/workflow): set theo số entry Changelog sau khi ghi entry mới: 1 → `low`, ≥2 → `medium`, ≥3 → `high`.

6. **Log**: Ghi vào `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] distill | Tạo mới wiki/<type>-<slug>.md
   ## [YYYY-MM-DD] distill | Cập nhật wiki/<type>-<slug>.md: <mô tả thay đổi>
   ## [YYYY-MM-DD] distill | Gộp <slug-a> + <slug-b> → <slug-mới>
   ## [YYYY-MM-DD] distill | Bỏ qua (archive) inbox/<slug>.md: <lý do>
   ```

7. **Dọn inbox items đã xử lý thành công**: move sang `archive/inbox/` (KHÔNG xoá cứng). Item đã được đúc kết vào wiki/skill/workflow nên không cần trong inbox nữa, nhưng giữ ở `archive/inbox/` để: (a) trace ngược candidate gốc đến page đã tạo, (b) khôi phục được nếu user thấy distill phân loại sai mà kho không có git.

## Quy tắc phân loại

Chọn loại dựa trên bản chất nội dung:

| Tiêu chí | Loại |
|---|---|
| Thao tác làm-theo được, có thể dùng lại cho task tương tự (được phép gắn domain) | **Skill** |
| Chuỗi bước, gắn domain, gọi nhiều skill, thay đổi khi đổi domain | **Workflow** |
| Tri thức, khái niệm, quyết định, cách xử lý sự cố | **Wiki page** |
| Không chắc, hoặc mới gặp lần đầu | Mặc định **Wiki page** (chờ tín hiệu lặp rồi promote) |

Tại sao mặc định wiki? Wiki page an toàn hơn: dễ tạo, dễ sửa, dễ gộp. KHÔNG ép mọi thứ thành skill ngay. Nhưng đừng để wiki thành nghĩa địa: khi một cách làm bị dùng lặp nhiều lần, NÂNG nó thành skill. Skill KHÔNG cần "độc lập hoàn toàn với context dự án": chấp nhận skill gắn domain MIỄN LÀ làm-theo-được và tái dùng cho task tương tự. Tiêu chí loại bỏ skill chỉ là: thao tác quá cụ thể, dùng một lần, không lặp lại.

## Cross-referencing

Sau khi tạo/cập nhật page, kiểm tra 3 điều:

1. Page mới có liên quan page cũ nào không? → thêm `related: [...]` trong frontmatter.
2. Body có mention page khác không? → thêm `[[slug]]` link.
3. Page cũ có cần reference ngược đến page mới không? → cập nhật page cũ.

Tại sao quan trọng? Cross-reference biến danh sách page rời rạc thành mạng lưới tri thức. Khi tra cứu 1 page, tự động thấy các page liên quan.

## Lưu ý

- Không tự ý xoá inbox item chưa được user duyệt.
- Không tạo page mới nếu có thể cập nhật page cũ.
- Khi gộp page, giữ lại tất cả thông tin có giá trị từ các page gốc.
- Khi sửa page, giữ nguyên phần không liên quan đến thay đổi.
