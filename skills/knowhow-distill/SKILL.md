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
| Không đáng lưu | **BỎ QUA**, xoá khỏi inbox | Thông tin quá cụ thể, không tái sử dụng |

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

2. **Cập nhật registry tương ứng**:
   - Tạo/xoá wiki page → cập nhật `wiki/index.md`
   - Tạo/cập nhật skill → cập nhật `skills/registry.md`
   - Tạo/cập nhật workflow → cập nhật `workflows/registry.md`

3. **Cross-referencing**:
   - Thêm `related: [...]` trong frontmatter nếu page mới liên quan page cũ.
   - Thêm `[[...]]` link trong body khi mention page khác.
   - Cập nhật page cũ thêm reference ngược đến page mới (nếu cần).

4. **Changelog**: Ghi dòng changelog cuối page bị thay đổi. Lấy nguồn từ `source_file` của inbox item (trỏ `raw/...`), KHÔNG trỏ `inbox/...` (inbox sẽ bị xoá ở mục 7).
   ```
   ## Changelog
   - YYYY-MM-DD: [mô tả thay đổi] (source: raw/YYYY-MM-DD-slug.md)
   ```

5. **Version**: Tăng version trong frontmatter nếu skill/workflow bị sửa.

5b. **Lifecycle metadata**:
   - `status`: page mới set `active`. Khi hành động là SỬA (thay cách cũ), set page/section cũ `status: deprecated`. Khi GỘP, page bị nuốt set `status: archived`.
   - `confidence` (chỉ wiki, không áp skill/workflow): set theo số entry Changelog sau khi ghi entry mới — 1 → `low`, ≥2 → `medium`, ≥3 → `high`.

6. **Log**: Ghi vào `wiki/log.md`:
   ```
   - [distill] Tạo mới [type]/[slug].md
   - [distill] Cập nhật [type]/[slug].md: [mô tả thay đổi]
   - [distill] Gộp [slug-a].md + [slug-b].md → [slug-mới].md
   - [distill] Bỏ qua inbox/[slug].md: [lý do]
   ```

7. **Xoá inbox items** đã xử lý thành công.

## Quy tắc phân loại

Chọn loại dựa trên bản chất nội dung:

| Tiêu chí | Loại |
|---|---|
| Chạy độc lập, tái sử dụng cao, không phụ thuộc context dự án | **Skill** |
| Chuỗi bước, gắn domain, gọi nhiều skill, thay đổi khi đổi domain | **Workflow** |
| Tri thức, khái niệm, quyết định, cách xử lý sự cố | **Wiki page** |
| Không chắc | Mặc định **Wiki page** |

Tại sao mặc định wiki? Wiki page an toàn hơn: dễ tạo, dễ sửa, dễ gộp. Sau khi tích lũy đủ, có thể promote lên skill/workflow. Ngược lại, tạo skill quá sớm khi chưa đủ hiểu biết sẽ tạo ra skill kém chất lượng.

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
