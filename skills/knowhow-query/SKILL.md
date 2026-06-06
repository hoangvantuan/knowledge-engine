---
name: knowhow-query
description: "Trả lời câu hỏi nhắm vào knowhow dự án: đọc index + grep page liên quan, tổng hợp câu trả lời kèm trích dẫn [[slug]]. Phát tín hiệu query-miss khi hỏi không trúng page sạch. Nếu câu trả lời đáng tái dùng, file ngược qua inbox (không ghi thẳng wiki). Trigger: 'tra knowhow', 'dự án có gì về X', 'so sánh...', 'knowhow query', khi user hỏi một câu nhắm vào tri thức đã tích luỹ."
---

# Knowhow Query

Trả lời câu hỏi từ knowhow đã tích luỹ. Đồng thời là nguồn tín hiệu thứ ba cho tiến hoá cấu trúc: câu hỏi lặp mà không trúng page là dấu hiệu mạnh của thiếu cấu trúc.

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user chạy `knowhow-init` trước.

## Phạm vi agent

`knowhow-query` và sản phẩm knowhow đều là markdown thuần, dùng chung cho mọi AI agent. Không gắn với agent cụ thể nào: mọi agent đều chạy được skill và đọc được kết quả query.

## Flow query

### Bước 1: Tìm page liên quan

1. Đọc `wiki/index.md` + `skills/registry.md` + `workflows/registry.md` để biết tổng quan.
2. Rút 3-5 từ khoá từ câu hỏi. Grep tìm page liên quan:
   ```bash
   grep -ril "<từ khoá>" .knowhow/wiki .knowhow/skills .knowhow/workflows
   ```
3. Đọc các file hit.

### Bước 2: Tổng hợp câu trả lời

- Tổng hợp từ các page đã đọc, trả lời trực tiếp câu hỏi.
- Trích dẫn nguồn bằng `[[slug]]` cho mỗi tuyên bố lấy từ một page cụ thể.
- Nếu không tìm thấy page nào liên quan, nói thẳng "chưa có knowhow về việc này" thay vì bịa.

### Bước 3: Phát tín hiệu query-miss

Đánh giá chất lượng câu trả lời. Emit `query-miss` vào `.knowhow/schema-signals.md` khi MỘT trong hai:

- Phải chắp vá từ **≥ 3 page** mới trả lời được (T = 3, khởi điểm).
- KHÔNG có page sạch nào trực tiếp trả lời (toàn suy luận chắp nối).

Chèn dòng tín hiệu NGAY DƯỚI heading `## Đang chờ xử lý` (KHÔNG append cuối file, vì cuối file là vùng `## Đã xử lý`):
```bash
awk '/^## Đang chờ xử lý$/{print; print "- [YYYY-MM-DD] query | query-miss | <câu hỏi rút gọn>, phải chắp <N> page | related: tag:<chủ-đề>"; next} 1' \
  .knowhow/schema-signals.md > /tmp/ss && mv /tmp/ss .knowhow/schema-signals.md
```

Ví dụ dòng tín hiệu:
```
- [2026-06-07] query | query-miss | "so sánh 3 thí nghiệm A/B/C", phải chắp 4 page | related: tag:experiment
```

> Nếu câu trả lời gọn (1-2 page sạch trúng đích), KHÔNG emit. Tín hiệu chỉ dành cho "khuôn không vừa".

### Bước 3b: Phát tín hiệu promote-candidate

Ngoài query-miss (khuôn không vừa), query là chỗ tốt để bắt "một page bị dùng đi dùng lại để LÀM THEO", ứng viên nâng thành skill.

Emit `promote-candidate` vào `.knowhow/schema-signals.md` khi CẢ HAI đúng:

- Câu hỏi có tính thao tác ("làm sao để...", "các bước...", "quy trình..."), và
- Câu trả lời dựa chủ yếu vào MỘT page type pattern/troubleshooting (page đó là nguồn hành động, không chỉ tham khảo).

Chèn dòng tín hiệu NGAY DƯỚI heading `## Đang chờ xử lý`. THAY `YYYY-MM-DD` bằng ngày thật và `<slug-page>` bằng slug thật TRƯỚC khi chạy:

```bash
awk '/^## Đang chờ xử lý$/{print; print "- [YYYY-MM-DD] query | promote-candidate | hỏi làm-theo trúng [[<slug-page>]] | related: <slug-page>"; next} 1' \
  .knowhow/schema-signals.md > /tmp/ss && mv /tmp/ss .knowhow/schema-signals.md
```

> Mỗi lần page đó lại được hỏi kiểu làm-theo thì emit thêm một dòng (cùng slug). distill đếm số dòng cùng slug để áp ngưỡng. KHÔNG dedupe ở đây: mỗi lần lặp là một phiếu.
> Page đã có skill tương ứng (grep cố định `grep -F 'promoted_from: [[<slug>]]' .knowhow/skills` ra kết quả) thì KHÔNG emit nữa.

### Bước 4: File ngược qua inbox (tôn trọng cửa duy nhất)

Nếu câu trả lời đáng tái dùng (user xác nhận OK), thả nó vào `inbox/` như một candidate page. KHÔNG ghi thẳng vào `wiki/`. distill xử lý sau như mọi item khác.

1. Đánh giá đáng lưu: câu trả lời đáng lưu nếu có thể tái dùng cho câu hỏi tương tự sau này (không phải thông tin quá cụ thể, dùng một lần). Nếu đáng, hỏi user: "Câu trả lời này đáng lưu lại không?"
2. Nếu user OK:
   - Lưu Q&A nguyên văn vào `raw/YYYY-MM-DD-query-<slug>.md` (provenance, nhất quán với quy tắc capture).
   - Tạo inbox item `inbox/YYYY-MM-DD-query-<slug>.md` theo format inbox trong `../knowhow-capture/references/page-formats.md` (đường dẫn tương đối từ thư mục skill này), với:
     - `captured_from: query`
     - `source_file: raw/YYYY-MM-DD-query-<slug>.md`
   - KHÔNG ghi vào `wiki/`, `skills/`, `workflows/`.

### Bước 5: Ghi log

Thêm vào `wiki/log.md`:
```
## [YYYY-MM-DD] query | <câu hỏi rút gọn> (emit query-miss: có/không, file inbox: có/không)
```

## Quy tắc cứng

1. query KHÔNG ghi thẳng vào `wiki/`. Mọi knowhow mới đi qua `inbox/` → distill (giữ bất biến "cửa duy nhất").
2. Không bịa câu trả lời. Không có page thì nói không có.
3. Trích dẫn `[[slug]]` cho mọi tuyên bố lấy từ page.
4. Chỉ emit `query-miss` khi thật sự "khuôn không vừa" (≥3 page hoặc không page sạch), tránh báo nhiễu.
5. Emit `promote-candidate` chỉ khi câu hỏi có tính làm-theo VÀ trả lời dựa chủ yếu vào một pattern/troubleshooting page. Không emit cho câu hỏi tra cứu thuần (để biết). Không emit nếu page đó đã được promote (đã có skill với `promoted_from` trỏ về).
