# Batch Ingest: Capture nguồn lớn hoặc nhiều nguồn

Reference cho `knowhow-capture` chế độ "Từ nguồn ngoài" khi nguồn vượt cỡ một lần đọc-duyệt thường: meeting transcript, export chat, loạt issue/PR, postmortem, tài liệu dài.

## Khi nào dùng batch

- Một file dài hơn khoảng 500 dòng, HOẶC từ 3 file trở lên trong cùng một lần capture.
- Dưới ngưỡng đó: dùng flow capture thường, không cần file này.

## Nguyên tắc giữ nguyên

Batch KHÔNG phá bất biến nào của hệ: vẫn chỉ ghi vào `inbox/`, vẫn user duyệt trước khi ghi. Khác biệt duy nhất là cách đọc (cuốn chiếu) và cách duyệt (theo nhóm thay vì từng item).

## Bước 1: Lưu nguồn gốc vào raw/ TRƯỚC khi đọc

- Copy nguyên văn file nguồn vào `raw/YYYY-MM-DD-<slug-nguồn>.md`. Một file nguồn = MỘT file raw, không tách per-item như capture thường.
- Mọi inbox item sinh từ nguồn này trỏ chung `source_file` về file raw đó. Vị trí cụ thể (số dòng, timestamp, link message) ghi trong "Context gốc" của từng item.

## Bước 2: Đọc theo bản đồ loại nguồn

Mỗi loại nguồn có "quặng" riêng. Tìm đúng chỗ, bỏ đúng thứ:

| Nguồn | Ưu tiên tìm | Chủ động bỏ qua |
|---|---|---|
| Meeting transcript | decision (điều được chốt), concept (thuật ngữ được định nghĩa), lesson (đoạn nhìn lại) | action item thuần (việc cần làm, không phải tri thức), small talk |
| Chat / Slack export | troubleshooting (thread fix lỗi), pattern (mẹo được share), decision | tin nhắn điều phối, link thả không ngữ cảnh |
| Issue / PR / postmortem | troubleshooting, lesson, decision | diễn biến xử lý từng comment |
| Tài liệu / báo cáo | concept, pattern, decision | số liệu chỉ đúng một thời điểm, không tái dùng |

## Bước 3: Đọc cuốn chiếu, gom xong mới trình

- Nguồn dài: đọc theo đoạn, gom candidates qua các đoạn vào MỘT danh sách. KHÔNG trình từng đoạn một (user sẽ phải duyệt rải rác).
- Dedupe trong batch: cùng một quyết định được nhắc 3 lần trong cuộc họp = 1 candidate.
- Đối chiếu nhanh với `wiki/index.md`: candidate có vẻ trùng tri thức đã có thì VẪN GIỮ, nhưng đánh dấu "(có thể trùng [[slug]], distill sẽ phân xử)". Capture không merge, đó là việc của distill.

## Bước 4: Trình duyệt theo nhóm

- Nhóm candidates theo loại (decision, lesson, troubleshooting...), mỗi nhóm tối đa khoảng 10 item.
- User thao tác theo nhóm: "giữ cả nhóm", "bỏ cả nhóm", "giữ trừ item N". Vẫn cho sửa từng item khi user muốn.
- Trên 30 candidates: trình đợt 1 gồm các loại giá trị cao trước (decision, lesson, troubleshooting), hỏi user có muốn xem tiếp đợt 2 (concept, pattern, candidate-skill/workflow) không.

## Bước 5: Ghi inbox + log

- Mỗi item được duyệt: `inbox/YYYY-MM-DD-<slug>.md` theo format chuẩn của page-formats, `source_file` trỏ file raw chung.
- Log một dòng cho cả batch:
  ```
  ## [YYYY-MM-DD] capture | Batch ingest <tên nguồn>: N items (M decision, K lesson, ...)
  ```
- Nguồn quá lớn phải chia nhiều phiên: ghi rõ phạm vi đã xử lý trong log ("phần 1/2, đến dòng 800" hoặc "đến timestamp 01:10:00") để phiên sau tiếp đúng chỗ, không ingest trùng.
