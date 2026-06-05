# Consolidation Checklist

8 hạng mục rà soát cho deep audit. Với mỗi hạng mục: đọc nội dung, phân tích, báo cáo kèm đề xuất hành động.

---

## 1. Nhất quán nội dung

**Mô tả**: Tìm chỗ các page nói khác nhau về cùng một vấn đề.

**Cách kiểm tra**:
- Đọc tất cả page, nhóm theo chủ đề (dựa trên `tags`, `related`, nội dung).
- So sánh các tuyên bố về cùng một vấn đề giữa các page.
- Chú ý đặc biệt: cấu hình, công cụ, quy trình, convention.

**Ví dụ**:
- Skill A nói "dùng PostgreSQL", wiki page B nói "dùng MySQL".
- Wiki page C nói "deploy bằng Docker", workflow D nói "deploy bằng K8s".

**Hành động**: Flag mâu thuẫn. Đề xuất page nào là source of truth. Hỏi user xác nhận trước khi sửa.

---

## 2. Nhất quán thuật ngữ

**Mô tả**: Tìm cùng khái niệm nhưng dùng nhiều tên hoặc viết tắt khác nhau.

**Cách kiểm tra**:
- Trích xuất tất cả thuật ngữ kỹ thuật, tên riêng, viết tắt từ các page.
- Nhóm các thuật ngữ giống nghĩa.
- Đánh dấu nhóm nào dùng không nhất quán.

**Ví dụ**:
- "API Gateway" vs "gateway" vs "APIGW" cho cùng một thành phần.
- "user story" vs "story" vs "US" cho cùng khái niệm.

**Hành động**: Đề xuất tên chuẩn cho mỗi nhóm. Sau khi user duyệt, find-replace trên tất cả page liên quan.

---

## 3. Tính hợp lệ

**Mô tả**: Skill và workflow reference công cụ, file, API, command cụ thể. Kiểm tra chúng còn hợp lệ không.

**Cách kiểm tra**:
- Trích xuất tên công cụ, đường dẫn file, API endpoint, command từ nội dung skill/workflow.
- Kiểm tra trong codebase: file có tồn tại? Command có chạy được? API còn dùng?
- So sánh với `package.json`, `requirements.txt`, hoặc config hiện tại.

**Ví dụ**:
- Skill reference `scripts/deploy.sh` nhưng file đã bị xoá.
- Workflow dùng command `npm run lint` nhưng project đã chuyển sang `pnpm`.

**Hành động**: Flag reference không hợp lệ. Đề xuất cập nhật hoặc archive page.

---

## 4. Gộp và tách

**Mô tả**: Tìm page nội dung chồng chéo đáng kể (cần gộp) hoặc page quá dài (cần tách).

**Cách kiểm tra**:
- So sánh nội dung giữa các page cùng category.
- Tìm đoạn văn trùng lặp hoặc nói cùng ý.
- Đếm số dòng mỗi page. Page > 200 dòng → ứng viên tách.

**Ví dụ**:
- `wiki/deployment.md` và `wiki/ci-cd.md` cùng giải thích pipeline deploy → gộp.
- `skills/data-migration.md` dài 350 dòng, cover cả planning, execution, rollback → tách thành 3.

**Hành động**: Đề xuất cụ thể page nào gộp thành page gì. Hoặc page nào tách thành những page con nào. Hỏi user trước khi thực thi.

---

## 5. Độ phủ

**Mô tả**: So sánh cấu trúc dự án với knowhow đã có. Tìm vùng chưa được cover.

**Cách kiểm tra**:
- Liệt kê thư mục và module chính trong dự án (đọc cấu trúc thư mục gốc).
- Liệt kê chủ đề đã có knowhow (dựa trên registry và tags).
- Tìm thư mục/module nào chưa có knowhow nào đề cập.

**Ví dụ**:
- Dự án có thư mục `auth/` nhưng không có wiki page nào về authentication.
- Module `payments` phức tạp nhưng chỉ có 1 wiki page sơ sài.

**Hành động**: Liệt kê lĩnh vực thiếu. Đề xuất tạo stub page cho lĩnh vực quan trọng nhất trước.

---

## 6. Chuỗi phụ thuộc

**Mô tả**: Workflow reference skill. Kiểm tra version skill có tương thích không.

**Cách kiểm tra**:
- Đọc mỗi workflow, trích xuất `[[skill-name]]` reference.
- Đọc skill được reference, lấy `version` từ frontmatter.
- So sánh: nếu skill đã tăng version kể từ lần workflow được tạo hoặc cập nhật gần nhất → flag.

**Ví dụ**:
- Workflow `deploy-production` reference `[[deploy-skill]]` tạo lúc skill version 1.0.
- `deploy-skill` hiện tại là version 2.0, đã thay đổi input format.

**Hành động**: Đề xuất user review workflow với skill version mới. Ghi rõ breaking changes nếu phát hiện.

---

## 7. Lỗi thời

**Mô tả**: Page không được cập nhật lâu ngày hoặc nội dung không còn phản ánh thực tế.

**Cách kiểm tra**:
- Page có `updated` hoặc `created` cũ hơn 30 ngày VÀ `confidence` không phải "high" → ứng viên.
- Skill/workflow version cũ mà dự án đã thay đổi nhiều (so sánh git log nếu có).
- Nội dung đề cập công nghệ/version đã cũ.

**Ví dụ**:
- Wiki page về "Node.js 16 setup" nhưng dự án đã chuyển sang Node.js 20.
- Skill "backup database" viết 3 tháng trước, database schema đã thay đổi nhiều.

**Hành động**: Đề xuất review (cập nhật nội dung) hoặc archive (chuyển vào `archive/`). Không tự xoá.

---

## 8. Trang mồ côi (orphan)

**Mô tả**: Page không được reference từ page nào khác và không nằm trong `related:` của page nào.

**Cách kiểm tra**:
- Xây graph liên kết: mỗi page là node, mỗi `[[...]]` và `related:` entry là edge.
- Tìm node không có edge vào (in-degree = 0).
- Loại trừ: index page, registry page (chúng là root node, không cần link vào).

**Ví dụ**:
- `wiki/old-architecture.md` không được link từ đâu, không nằm trong `related:` nào.
- `skills/deprecated-build.md` tồn tại trong registry nhưng không page nào reference.

**Hành động**: Đề xuất thêm link từ page liên quan. Hoặc review xem page có còn cần thiết không. Nếu không cần → archive.
