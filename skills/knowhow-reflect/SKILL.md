---
name: knowhow-reflect
description: "Phỏng vấn phản tư có cấu trúc để moi bài học: retro, retrospective, AAR, post-mortem, rút kinh nghiệm sau sự cố. Tín hiệu cốt lõi: user vừa trải qua việc đáng kể (sự cố, downtime, dự án trễ, migration đau, sprint) nhưng bài học còn nằm trong đầu, chưa thành chữ. Họ muốn ĐƯỢC PHỎNG VẤN, không chỉ được nhắc ngồi viết. Khác knowhow-capture (ghi cái đã nói ra trong hội thoại): skill này moi tri thức ngầm trước bằng hỏi-tới-gốc, rồi bàn giao cho capture. Dùng khi user nói 'muốn moi bài học', 'làm AAR', 'rút kinh nghiệm từ', hoặc kể một trải nghiệm vừa rồi và muốn học từ nó một cách hệ thống."
---

# Knowhow Reflect

Phỏng vấn phản tư để rút tri thức ngầm thành chữ. Khác `knowhow-capture` ở một điểm bản chất: capture **nhặt** cái đã được nói ra trong hội thoại; reflect **moi** cái còn nằm trong đầu người, chưa từng thành chữ.

Reflect KHÔNG ghi file. Nó là tầng phỏng vấn đặt trước capture: phỏng vấn xong, tổng hợp candidates, rồi chạy flow `knowhow-capture` từ bước trình danh sách. Bất biến "một cửa inbox" giữ nguyên.

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user chạy `knowhow-init` trước.

## Khi nào dùng

| Bối cảnh | Tín hiệu | Trọng tâm phỏng vấn |
|---|---|---|
| Sau sự cố | Vừa xử lý xong một lỗi, một khủng hoảng | Nguyên nhân gốc + phòng ngừa |
| Sau milestone / cuối dự án | Bàn giao xong, kết thúc một giai đoạn | Kỳ vọng vs thực tế + bài học |
| Định kỳ (tuần / sprint) | User muốn retro | Việc lặp lại đáng đúc kết + lỗi lặp |

Nếu user không nói rõ bối cảnh, hỏi một câu: "Phản tư về việc gì: sự cố vừa rồi, dự án vừa xong, hay tuần vừa qua?"

## Kịch bản phỏng vấn (5 câu lõi)

Hỏi TỪNG CÂU MỘT, chờ trả lời rồi mới hỏi tiếp. Đào sâu theo câu trả lời, không đọc cả danh sách một lượt.

1. **Kỳ vọng**: Ban đầu định đạt gì? Tiêu chí thành công lúc đó là gì?
2. **Thực tế**: Kết quả thật ra sao? Lệch chỗ nào so với kỳ vọng (cả tốt hơn lẫn xấu hơn)?
3. **Nguyên nhân gốc**: Tại sao lệch? Hỏi "tại sao" tiếp cho đến gốc, tối thiểu 3 lớp. Tách rõ nguyên nhân trực tiếp và điều kiện bối cảnh xung quanh.
4. **Bài học**: Nói trong MỘT câu cho người ngoài cuộc hiểu. Bài học này áp dụng khi nào, và khi nào thì KHÔNG áp dụng?
5. **Hành động hệ thống**: Để lần sau không phụ thuộc trí nhớ, cái gì trong kho phải đổi? (thêm mục vào workflow nào, sửa skill nào, tạo quyết định nào)

## Quy tắc phỏng vấn

- **Gợi nhớ bằng dữ kiện, không hỏi chay.** Trước khi hỏi, đọc nguồn sẵn có (hội thoại hiện tại, `wiki/log.md`, transcript/file user đưa) và nhắc lại mốc cụ thể: "Hôm X mình gặp lỗi Y, lúc đó...". Trí nhớ con người bám sự kiện, không bám câu hỏi trừu tượng.
- **Không phán xét.** Mục tiêu là rút bài học, không phải quy trách nhiệm. Câu trả lời "do tôi quên" phải được đào tiếp: "vì sao quên được, hệ thống thiếu chốt chặn nào?"
- **Một lần phản tư, một chủ đề.** Nhiều sự việc thì hẹn vòng sau, không tham.
- **Ngắn.** Mục tiêu 10-15 phút. Đủ 5 câu là dừng, không kéo dài.
- **User trả lời nông thì chấp nhận.** Ghi nhận mức user đưa, không ép. Bài học nông vẫn hơn không có gì; vòng distill sau sẽ làm dày thêm.

## Flow

### Bước 1: Chọn bối cảnh + đọc dữ kiện

Xác định bối cảnh (bảng trên). Đọc nguồn liên quan để chuẩn bị mồi gợi nhớ.

### Bước 2: Phỏng vấn

Chạy kịch bản 5 câu, từng câu một, đào sâu theo quy tắc trên.

### Bước 3: Tổng hợp candidates

Từ nội dung phỏng vấn, tách thành các candidate theo tiêu chí nhận diện của `knowhow-capture`. Một phiên reflect điển hình sinh ra:

- 1 item `lesson` (lõi của phiên: kỳ vọng, thực tế, nguyên nhân gốc, bài học, hành động hệ thống).
- 0-n item `decision` / `pattern` / `troubleshooting` / `concept` nếu lộ ra trong lúc đào.
- 0-n item `candidate-skill` / `candidate-workflow` từ câu hỏi số 5 (hành động hệ thống).

### Bước 4: Bàn giao cho capture

Chạy flow `knowhow-capture` từ Bước 2 (trình danh sách candidates cho user duyệt). Khác biệt duy nhất:

- `captured_from: reflect` trong frontmatter inbox item.
- Phần "Context gốc" trích nguyên văn lời user trả lời phỏng vấn (đây chính là tri thức ngầm vừa thành chữ, giá trị provenance cao nhất).
- Nguồn lưu `raw/`: ghi tóm tắt hỏi-đáp của phiên phỏng vấn vào `raw/YYYY-MM-DD-reflect-<slug>.md`.

Capture lo phần còn lại: duyệt, ghi inbox, ghi log, gợi ý distill.

## Quy tắc cứng

1. Reflect KHÔNG ghi vào `wiki/`, `skills/`, `workflows/`, và KHÔNG tự ghi `inbox/`. Mọi kết quả đi qua flow `knowhow-capture`.
2. Hỏi từng câu một. CẤM dán cả 5 câu thành một bảng hỏi.
3. Câu trả lời dừng ở triệu chứng hoặc đổ lỗi cá nhân thì hỏi tiếp "tại sao", tối thiểu 3 lớp.
4. Câu số 5 (hành động hệ thống) là bắt buộc. Bài học không kèm thay đổi hệ thống là bài học sẽ bị quên.
5. Trích nguyên văn lời user trong "Context gốc". Không diễn đạt lại làm mất giọng gốc.
