## Knowhow

Dự án này sử dụng hệ thống Knowhow để quản lý tri thức và kỹ năng.

### Bản đồ tri thức (nạp tự động đầu phiên)

4 file dưới đây là "bản đồ" để định tuyến công việc. hãy ĐỌC đủ 4 file trên ngay khi bắt đầu phiên, trước khi làm việc.

.knowhow/SCHEMA.md
.knowhow/wiki/index.md
.knowhow/skills/registry.md
.knowhow/workflows/registry.md

Chỉ nạp 4 file bản đồ này, KHÔNG nạp sẵn nội dung chi tiết. Wiki page, skill và workflow bó được load on-demand qua `knowhow-query` / `knowhow-run` khi cần.

### Quy trình làm việc

1. Khi gặp vấn đề, tra `.knowhow/wiki/index.md` trước
2. Khi cần thực hiện quy trình, tra `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md` (match task theo cột `Khi nào dùng`, `Mô tả`, `Tags`)
3. Khi đã tìm được skill/workflow phù hợp, dùng `knowhow-run` để load file bó và làm theo, KHÔNG chỉ liếc một dòng mô tả rồi tự làm
4. Sau phiên làm việc có bài học đáng ghi nhận, đề xuất capture vào `.knowhow/inbox/`
5. Khi `.knowhow/inbox/` có ≥ 5 item hoặc có item cũ hơn 7 ngày, chủ động nhắc user chạy `knowhow-distill` để đúc kết, tránh inbox tồn đọng.
