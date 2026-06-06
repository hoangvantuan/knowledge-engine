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
grep -rh "^tags:" .knowhow/wiki | sed 's/tags://' | tr -d '[]' | tr ',' '\n' | sed 's/^[[:space:]]*//; s/[[:space:]]*$//' | grep -v '^$' | sort | uniq -c | sort -rn
# Orphan ratio: page không được [[link]] hay related trỏ tới (tham khảo consolidation checklist mục 8)
```

> Lưu ý: quét tag giả định format inline `tags: [a, b]` (chuẩn của page-formats.md). Vòng lặp đếm type ở trên nên bao gồm cả type mới đã thêm vào SCHEMA Page Types, không chỉ 4 base type.

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
3. Cập nhật vòng lặp quét sống ở mục 2 để bao gồm type mới (thêm vào danh sách `for t in ...`).
4. (Tín hiệu liên quan: `no-fit-type`, `tag-cluster`, `query-miss` cùng chủ đề.)

### 4.2. Đổi layout (subfolder)

1. Tạo subfolder `wiki/<type>/` (hoặc nhóm theo tag).
2. Move các file của type đó vào subfolder.
3. **Rewrite mọi `[[link]]`** trỏ tới (xem grep ở Bước chung).
4. Cập nhật ghi chú resolve trong `SCHEMA.md` (mục Cross-referencing đã nói resolve tìm đệ quy, xác nhận còn đúng).

### 4.3. Đổi format page type

1. Thêm section vào template type trong `../../knowhow-capture/references/page-formats.md` (đường dẫn tương đối từ thư mục chứa file này, `skills/knowhow-lint/references/`) (áp cho page tạo MỚI).
2. Backfill page cũ là **TUỲ CHỌN**: đề xuất riêng từng đợt, KHÔNG tự động hàng loạt.
3. (Tín hiệu liên quan: `adhoc-section`.)

### 4.4. Thêm mục SCHEMA

1. Sửa text `SCHEMA.md`: thêm thuật ngữ vào mục "Glossary & Convention" hoặc quy ước vào "Quy tắc vận hành".

### Type nghỉ hưu (retire)

Page của type bị nghỉ hưu: set `status: archived`, move vào `archive/`. Xoá định nghĩa type khỏi bảng Page Types.

## 5. Bước chung cuối mỗi batch migrate

Bắt buộc, theo đúng thứ tự:

1. **Bump `schema_version`** ở đầu `SCHEMA.md` (tăng 1).
2. **Ghi Changelog SCHEMA**: thêm dòng `- YYYY-MM-DD: <loại thay đổi>, <lý do ngắn>` vào mục `## Changelog`.
3. **Rewrite link ảnh hưởng**: grep + sửa mọi `[[old-slug]]`:
   ```bash
   grep -rln "\[\[<old-slug>\]\]" .knowhow
   ```
4. **Rebuild index**: chạy mode `rebuild-index` (sinh lại `wiki/index.md` + 2 registry từ frontmatter).
5. **Ghi log**: `## [YYYY-MM-DD] lint | Schema-review: <loại>, <N> file migrate`.
6. **Đánh dấu tín hiệu đã xử lý**: cắt các dòng tín hiệu liên quan từ "Đang chờ xử lý" sang "Đã xử lý" trong `schema-signals.md` (để không đếm lại lần sau).

> **Reversibility**: toàn bộ là markdown trong git. Nếu migrate sai, `git revert` đưa về trạng thái cũ sạch.

## 6. Phụ thuộc bắt buộc (không bỏ sót)

- **Tái dùng** `rebuild-index` làm bước cuối mỗi migrate.
- **Phải mở rộng** resolve `[[slug]]` cho type mới + subfolder TRƯỚC khi migrate sinh chúng. Nếu không, lint báo link hỏng giả. (Xem `knowhow-lint/SKILL.md` mục 1b.)
- Type nghỉ hưu dùng `status: archived` + `archive/`.
