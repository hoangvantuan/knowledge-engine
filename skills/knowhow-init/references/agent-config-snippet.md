<!-- knowhow:start -->
## Knowhow

Kho tri thức này dùng hệ thống **Knowhow** để quản trị tri thức tích luỹ: thu thập tri thức sinh ra trong lúc làm việc, đúc kết thành dạng có cấu trúc, tái sử dụng cho việc sau, và để khuôn tự tiến hoá khi kho lớn lên. Mục tiêu là không để tri thức biến mất sau mỗi phiên, biến không gian làm việc thành một kho tích luỹ có cấu trúc. AI viết, người duyệt.

### Bản đồ tri thức

@.knowhow/SCHEMA.md
@.knowhow/skills/registry.md
@.knowhow/wiki/index.md
@.knowhow/workflows/registry.md

Đây là 4 file "bản đồ" để định tuyến công việc. KHÔNG nạp sẵn nội dung chi tiết; wiki page, skill và workflow bó được load on-demand qua `knowhow-query` / `knowhow-run` khi cần.

### Quy trình làm việc

1. Khi gặp vấn đề, tra `.knowhow/wiki/index.md` trước
2. Khi cần thực hiện quy trình, tra `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md` (match task theo cột `Khi nào dùng`, `Mô tả`, `Tags`)
3. Khi đã tìm được skill/workflow phù hợp, dùng `knowhow-run` để load file bó và làm theo, KHÔNG chỉ liếc một dòng mô tả rồi tự làm
4. Sau phiên làm việc có bài học đáng ghi nhận, đề xuất capture vào `.knowhow/inbox/`
5. Sau khi xử lý xong một sự cố, hoặc khi user nói vừa kết thúc một dự án/giai đoạn, đề xuất chạy `knowhow-reflect` (phỏng vấn rút bài học). Tri thức ngầm không tự thành chữ nếu không ai hỏi.
6. Khi `.knowhow/inbox/` có ≥ 5 item hoặc có item cũ hơn 7 ngày, chủ động nhắc user chạy `knowhow-distill` để đúc kết, tránh inbox tồn đọng.
<!-- knowhow:end -->
