---
name: knowhow-query
description: "Dùng khi 'kho' hoặc 'knowhow' là ĐÍCH được hỏi: user hỏi kho tri thức chứa gì, không phải hỏi web hay codebase. Bao gồm: tra bài học/quyết định của team ('kho có gì về X', 'kho tri thức có...không'), tìm cách làm đã ghi ('tìm/so sánh trong knowhow'), tổng hợp từ knowhow đã tích luỹ trong kho, hoặc soạn tài liệu onboarding/đào tạo từ wiki của kho. Tín hiệu phân biệt: câu hỏi hướng VÀO kho như nguồn thông tin. KHÔNG dùng cho: tìm trên internet, đọc file project/log/config bất kỳ, làm theo một quy trình đã đúc kết (đó là knowhow-run), hay kiểm tra sức khoẻ hệ thống (đó là knowhow-lint)."
---

# Knowhow Query

Trả lời câu hỏi từ knowhow đã tích luỹ. Đồng thời là nguồn tín hiệu thứ ba cho tiến hoá cấu trúc: câu hỏi lặp mà không trúng page là dấu hiệu mạnh của thiếu cấu trúc.

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user chạy `knowhow-init` trước.

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

Ghi theo [schema-signals-protocol.md](../knowhow-distill/references/schema-signals-protocol.md). Dòng query-miss có dạng `- [YYYY-MM-DD] query | query-miss | <câu hỏi rút gọn>, phải chắp <N> page | related: tag:<chủ-đề>`.

> Nếu câu trả lời gọn (1-2 page sạch trúng đích), KHÔNG emit. Tín hiệu chỉ dành cho "khuôn không vừa".

### Bước 3b: Phát tín hiệu promote-candidate

Ngoài query-miss (khuôn không vừa), query là chỗ tốt để bắt "một page bị dùng đi dùng lại để LÀM THEO", ứng viên nâng thành skill.

Emit `promote-candidate` vào `.knowhow/schema-signals.md` khi CẢ HAI đúng:

- Câu hỏi có tính thao tác ("làm sao để...", "các bước...", "quy trình..."), và
- Câu trả lời dựa chủ yếu vào MỘT page type pattern/troubleshooting (page đó là nguồn hành động, không chỉ tham khảo).

Ghi theo [schema-signals-protocol.md](../knowhow-distill/references/schema-signals-protocol.md) (quy tắc "1 phiếu mỗi lần, không dedupe" và "đã promote thì thôi" nằm ở đó). Dòng promote-candidate có dạng `- [YYYY-MM-DD] query | promote-candidate | hỏi làm-theo trúng [[<slug-page>]] | related: <slug-page>`.

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

## Chế độ teach (soạn lộ trình truyền đạt)

Kích hoạt khi user yêu cầu kiểu "soạn tài liệu onboarding", "dạy lại X cho người mới", "tổng hợp cho người khác học". Bản chất: vẫn là query (đọc wiki, tổng hợp, trích dẫn), khác ở hình thức output là một LỘ TRÌNH ĐỌC thay vì một câu trả lời.

1. **Gom**: như Bước 1 query thường, theo chủ đề/vai trò user nêu.
2. **Xếp theo thang biết → hiểu → làm**:
   - Mở đầu: `concept` (từ vựng phải biết trước).
   - Giữa: `decision` + `lesson` (vì sao mọi thứ như hiện tại, bẫy đã gặp).
   - Cuối: `pattern` + skill/workflow (làm thế nào, theo bó nào).
3. **Soạn lộ trình**: mỗi mục gồm tóm tắt 2-3 câu + link `[[slug]]` đến bản đầy đủ + 1 câu hỏi tự kiểm. KHÔNG copy nguyên văn page vào lộ trình: wiki vẫn là bản gốc duy nhất, lộ trình chỉ là bản đồ đọc. Copy nguyên văn sẽ tạo bản sao lỗi thời dần, đúng cái hệ này chống.
4. **Đầu ra**: trình cho user. Chỉ ghi ra file khi user chỉ định nơi lưu, và lưu NGOÀI `.knowhow/` (ví dụ `docs/onboarding-<chủ-đề>.md`). KHÔNG ghi vào `wiki/`.
5. **Lỗ hổng lộ ra khi soạn** (chủ đề quan trọng không có page, chuỗi học đứt quãng): báo user kèm danh sách thiếu, gợi ý capture/distill bổ sung. Mục cơ bản mà phải chắp ≥3 page: emit `query-miss` như thường (Bước 3).
6. **Log**: `## [YYYY-MM-DD] query | teach: <chủ đề> (N page, thiếu: <danh sách ngắn hoặc "không">)`.

## Quy tắc cứng

1. query KHÔNG ghi thẳng vào `wiki/`. Mọi knowhow mới đi qua `inbox/` → distill (giữ bất biến "cửa duy nhất").
2. Không bịa câu trả lời. Không có page thì nói không có.
3. Trích dẫn `[[slug]]` cho mọi tuyên bố lấy từ page.
4. Chỉ emit `query-miss` khi thật sự "khuôn không vừa" (≥3 page hoặc không page sạch), tránh báo nhiễu.
5. Emit `promote-candidate` chỉ khi câu hỏi có tính làm-theo VÀ trả lời dựa chủ yếu vào một pattern/troubleshooting page. Không emit cho câu hỏi tra cứu thuần (để biết). Không emit nếu page đó đã được promote (đã có skill với `promoted_from` trỏ về).
