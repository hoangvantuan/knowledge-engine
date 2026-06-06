## Knowhow

Dự án này sử dụng hệ thống Knowhow để quản lý tri thức và kỹ năng.

1. Đọc `.knowhow/SCHEMA.md` trước khi bắt đầu làm việc
2. Khi gặp vấn đề, tra `.knowhow/wiki/index.md` trước
3. Khi cần thực hiện quy trình, tra `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md` (match task theo cột `Khi nào dùng`, `Mô tả`, `Tags`)
4. Khi đã tìm được skill/workflow phù hợp, dùng `knowhow-run` để load file bó và làm theo, KHÔNG chỉ liếc một dòng mô tả rồi tự làm
5. Sau phiên làm việc có bài học đáng ghi nhận, đề xuất capture vào `.knowhow/inbox/`
6. Khi `.knowhow/inbox/` có ≥ 5 item hoặc có item cũ hơn 7 ngày, chủ động nhắc user chạy `knowhow-distill` để đúc kết, tránh inbox tồn đọng.
