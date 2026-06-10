# Page Formats: Knowhow System

Tài liệu tham chiếu chứa tất cả format page cho toàn bộ Knowhow System.
Dùng chung cho `knowhow-capture` (ghi inbox) và `knowhow-distill` (đúc kết wiki/skill/workflow).

Format dưới đây dùng chung cho MỌI lĩnh vực: code hay không code đều cùng một cấu trúc, chỉ khác nội dung. Ví dụ minh hoạ cố ý lấy từ nhiều nghề (kỹ thuật, kinh doanh, nội dung, vận hành) để thấy khuôn áp được ở đâu cũng được.

## Mục lục

1. **Inbox Item Format** — format item ghi vào `inbox/`
2. **Wiki Page Formats** — 2.1 Decision · 2.2 Pattern · 2.3 Concept · 2.4 Troubleshooting · 2.5 Lesson · 2.6 Project (tuỳ chọn)
3. **Skill Page Format** — frontmatter + body cho skill đã đúc kết
4. **Workflow Page Format** — frontmatter + body cho workflow đã đúc kết
5. **Changelog Format** — dòng changelog cuối mỗi page
6. **Registry Formats** — 6.1 Skill Registry · 6.2 Workflow Registry

---

## 1. Inbox Item Format

Mỗi item capture ghi vào `inbox/YYYY-MM-DD-slug.md`.

### Frontmatter

```yaml
---
type: decision | pattern | troubleshooting | concept | lesson | candidate-skill | candidate-workflow
title: "Tên ngắn gọn"
tags: []
captured_from: conversation | file-import | query | run | reflect
captured_at: "YYYY-MM-DDTHH:mm"
source_file: "raw/YYYY-MM-DD-slug.md"  # BẮT BUỘC. Trích đoạn nguồn (cả hội thoại) luôn lưu vào raw/. Không để trống.
promote_of: ""  # TUỲ CHỌN. Chỉ điền khi item là ứng viên promote: slug wiki page nguồn (ví dụ pattern-retry-jitter). distill đọc page này rồi đề xuất tạo skill giữ liên kết ngược.
---
```

### Body

```markdown
## Tóm tắt

[1-3 câu mô tả nội dung chính]

## Chi tiết

[Nội dung chi tiết, bao gồm lý do, cách làm, bối cảnh]

## Context gốc

[Trích dẫn nguyên văn từ nguồn. Giữ nguyên để trace.]
```

### Ví dụ: Inbox item loại decision

```markdown
---
type: decision
title: "Chọn PostgreSQL thay MongoDB"
tags: [database, architecture]
captured_from: conversation
captured_at: "2026-06-05T11:30"
source_file: "raw/2026-06-05-chon-postgresql.md"
---

## Tóm tắt

Chọn PostgreSQL làm database chính thay vì MongoDB vì cần transaction ACID cho module thanh toán.

## Chi tiết

MongoDB ban đầu được cân nhắc vì schema linh hoạt. Tuy nhiên module thanh toán yêu cầu transaction ACID xuyên nhiều bảng. PostgreSQL hỗ trợ native, MongoDB chỉ có multi-document transaction từ 4.0 với hiệu năng kém hơn.

Trade-off: mất tính linh hoạt schema nhưng đổi lại data integrity cho phần critical nhất.

## Context gốc

> "Mình quyết định chuyển sang PostgreSQL. MongoDB multi-doc transaction quá chậm cho payment flow, test cho thấy latency tăng 3x so với Postgres."
```

---

## 2. Wiki Page Formats

### 2.1. Decision

File: `wiki/decision-slug.md`

#### Frontmatter

```yaml
---
type: decision
title: "Tên quyết định"
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
status: active   # active | deprecated | archived
---
```

#### Body

```markdown
## Bối cảnh

[Tại sao phải quyết định? Vấn đề gì cần giải quyết?]

## Các lựa chọn đã xét

| Lựa chọn | Ưu | Nhược |
|-----------|-----|-------|
| A | ... | ... |
| B | ... | ... |

## Quyết định

[Chọn gì? Mô tả rõ ràng.]

## Lý do

[Tại sao chọn phương án này? Tiêu chí quyết định là gì?]

## Hệ quả

[Kết quả mong đợi. Rủi ro chấp nhận. Ràng buộc mới.]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

#### Ví dụ

```markdown
---
type: decision
title: "Chọn kênh bán hàng chính cho dòng sản phẩm mới"
tags: [sales, kenh-phan-phoi, chien-luoc]
related: [dinh-vi-san-pham-moi]
created: 2026-06-06
updated: 2026-06-06
confidence: medium
status: active
---

## Bối cảnh

Dòng sản phẩm mới sắp ra mắt, ngân sách marketing có hạn. Phải dồn lực vào một kênh bán chính trong quý đầu thay vì dàn mỏng nhiều kênh.

## Các lựa chọn đã xét

| Lựa chọn | Ưu | Nhược |
|-----------|-----|-------|
| Sàn TMĐT (Shopee/Lazada) | Có sẵn lưu lượng, thanh toán/vận chuyển lo hết | Phí sàn cao, khó dựng thương hiệu |
| Website tự vận hành | Sở hữu khách hàng, biên lợi nhuận tốt | Phải tự kéo traffic, chi phí quảng cáo lớn lúc đầu |
| Bán qua đại lý/cộng tác viên | Mở rộng nhanh, tận dụng quan hệ sẵn | Khó kiểm soát giá và hình ảnh thương hiệu |

## Quyết định

Chọn sàn TMĐT làm kênh chính cho quý đầu, song song dựng website ở chế độ chờ.

## Lý do

Mục tiêu quý đầu là kiểm chứng nhu cầu thật, không phải tối đa biên lợi nhuận. Sàn cho lưu lượng có sẵn nên test được thông điệp và giá nhanh nhất với rủi ro thấp nhất. Website giữ lại để chuyển dần khi đã biết tệp khách nào mua.

## Hệ quả

- Biên lợi nhuận quý đầu thấp vì phí sàn, chấp nhận để đổi lấy dữ liệu nhu cầu.
- Phải đầu tư ảnh/nội dung gian hàng đạt chuẩn sàn ngay từ đầu.
- Khi có đủ khách quay lại, chủ động kéo về website để sở hữu tệp khách.

## Changelog

- 2026-06-06: Tạo mới từ inbox
```

### 2.2. Pattern

File: `wiki/pattern-slug.md`

#### Frontmatter

```yaml
---
type: pattern
title: "Tên pattern"
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
status: active   # active | deprecated | archived
---
```

#### Body

```markdown
## Bối cảnh

[Khi nào áp dụng pattern này? Điều kiện tiên quyết?]

## Nội dung

[Cách làm chi tiết, từng bước nếu cần]

## Ví dụ

[Ví dụ từ thực tế, kèm code nếu phù hợp]

## Anti-pattern

[Cách làm SAI thường gặp liên quan, nếu có]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

#### Ví dụ

```markdown
---
type: pattern
title: "Retry với exponential backoff và jitter"
tags: [resilience, api, network]
related: [circuit-breaker-pattern]
created: 2026-06-05
updated: 2026-06-05
confidence: high
status: active
---

## Bối cảnh

Khi gọi external API có thể fail tạm thời (network glitch, rate limit, server overload). Cần retry nhưng tránh thundering herd.

## Nội dung

1. Retry tối đa 3 lần.
2. Delay giữa các lần: `min(base * 2^attempt + random_jitter, max_delay)`.
3. `base` = 1s, `max_delay` = 30s, `jitter` = random 0-1s.
4. Chỉ retry với lỗi transient (5xx, timeout). KHÔNG retry 4xx.

## Ví dụ

```python
import time, random

def retry_with_backoff(fn, max_retries=3, base=1, max_delay=30):
    for attempt in range(max_retries):
        try:
            return fn()
        except TransientError:
            if attempt == max_retries - 1:
                raise
            delay = min(base * (2 ** attempt) + random.random(), max_delay)
            time.sleep(delay)
```

## Anti-pattern

- Retry ngay lập tức không delay: gây thundering herd, làm server quá tải thêm.
- Retry tất cả mọi lỗi: 401/403/404 retry là vô nghĩa.

## Changelog

- 2026-06-05: Tạo mới từ inbox
```

### 2.3. Concept

File: `wiki/concept-slug.md`

#### Frontmatter

```yaml
---
type: concept
title: "Tên khái niệm"
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
status: active   # active | deprecated | archived
---
```

#### Body

```markdown
## Định nghĩa

[Nghĩa của khái niệm trong bối cảnh của kho. Không phải định nghĩa sách giáo khoa chung chung.]

## Liên quan

[Mối quan hệ với các concept/decision/pattern khác]

## Ví dụ

[Ví dụ cụ thể trong kho]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

#### Ví dụ

```markdown
---
type: concept
title: "Khách hạng A (định nghĩa nội bộ công ty)"
tags: [sales, phan-khuc-khach, thuat-ngu-noi-bo]
related: [chinh-sach-cham-soc-vip, quy-trinh-phan-bo-sales]
created: 2026-06-06
updated: 2026-06-06
confidence: medium
status: active
---

## Định nghĩa

"Khách hạng A" trong công ty là khách có doanh số 12 tháng gần nhất từ 200 triệu trở lên VÀ đã mua lặp lại ít nhất 2 lần. Đây là định nghĩa nội bộ, không trùng với "khách VIP" chung chung ngoài thị trường.

Lưu ý: chỉ doanh số cao mà mua một lần thì KHÔNG tính hạng A (xếp vào "khách lớn vãng lai"), vì tiêu chí hạng A nhấn mạnh tính trung thành chứ không chỉ giá trị đơn.

## Liên quan

- [[chinh-sach-cham-soc-vip]]: Chỉ khách hạng A mới được áp dụng chính sách này.
- [[quy-trinh-phan-bo-sales]]: Khách hạng A được giao cho sales senior, không qua đội mới.

## Ví dụ

Công ty X mua 350 triệu trong năm qua nhưng chỉ một đơn duy nhất, KHÔNG phải hạng A. Chị Lan mua 240 triệu chia 4 lần trong năm, là hạng A. Cùng mức doanh số không đồng nghĩa cùng hạng.

## Changelog

- 2026-06-06: Tạo mới từ inbox
```

### 2.4. Troubleshooting

File: `wiki/troubleshooting-slug.md`

#### Frontmatter

```yaml
---
type: troubleshooting
title: "Tên sự cố"
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
status: active   # active | deprecated | archived
---
```

#### Body

```markdown
## Triệu chứng

[Biểu hiện bên ngoài. User/hệ thống thấy gì?]

## Nguyên nhân gốc

[Root cause thực sự, không phải triệu chứng]

## Cách fix

[Từng bước cụ thể]

1. Bước 1
2. Bước 2
3. ...

## Phòng ngừa

[Làm gì để không tái diễn?]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

#### Ví dụ

```markdown
---
type: troubleshooting
title: "Bài đăng fanpage bị tụt reach đột ngột"
tags: [social-media, facebook, reach, van-hanh]
related: [lich-dang-bai-toi-uu]
created: 2026-06-06
updated: 2026-06-06
confidence: medium
status: active
---

## Triệu chứng

- Reach trung bình mỗi bài tụt từ ~8.000 xuống ~1.500 trong vòng một tuần.
- Lượt tương tác (like, comment) giảm theo, dù nội dung không đổi.
- Không có cảnh báo vi phạm nào từ nền tảng.

## Nguyên nhân gốc

Nhóm vận hành bắt đầu chèn link ngoài (web bán hàng) ngay trong nội dung bài đăng. Thuật toán Facebook giảm phân phối các bài kéo người dùng rời khỏi nền tảng. Triệu chứng "tụt reach" chỉ là biểu hiện, gốc là hành vi đặt link trong body.

## Cách fix

1. Bỏ link ngoài khỏi nội dung chính của bài.
2. Đưa link xuống comment đầu tiên hoặc dùng nút "Tìm hiểu thêm".
3. Ưu tiên định dạng nền tảng đang đẩy (video ngắn/Reels) cho 3-5 bài kế tiếp.
4. Theo dõi reach 7 ngày để xác nhận hồi phục.

## Phòng ngừa

- Quy ước nội bộ: không đặt link ngoài trong body bài đăng, luôn để ở comment.
- Checklist trước khi đăng: kiểm tra định dạng, vị trí link, độ dài caption.
- Mỗi tháng rà thay đổi thuật toán của nền tảng để cập nhật quy ước.

## Changelog

- 2026-06-06: Tạo mới từ inbox
```

### 2.5. Lesson

File: `wiki/lesson-slug.md`

Khác `pattern` (cách làm đã chứng minh) và `troubleshooting` (sự cố + cách fix): `lesson` giữ cấu trúc phản tư, lõi là độ lệch giữa kỳ vọng và thực tế cùng hành động hệ thống rút ra. Thường sinh từ `knowhow-reflect`.

#### Frontmatter

```yaml
---
type: lesson
title: "Tên bài học"
tags: []
related: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
status: active   # active | deprecated | archived
---
```

#### Body

```markdown
## Bối cảnh

[Sự việc gì? Dự án/giai đoạn nào? Ai liên quan?]

## Kỳ vọng vs Thực tế

[Ban đầu định đạt gì. Thực tế ra sao. Lệch ở đâu, mức nào, số cụ thể nếu có.]

## Nguyên nhân gốc

[Chuỗi "tại sao" đến gốc. Tách nguyên nhân trực tiếp và điều kiện bối cảnh.]

## Bài học

[MỘT câu, người ngoài cuộc đọc hiểu được.]

## Khi áp dụng / Khi KHÔNG áp dụng

[Điều kiện bài học đúng. Điều kiện bài học không còn đúng.]

## Hành động hệ thống

[Cái gì trong kho phải đổi để lần sau không phụ thuộc trí nhớ: link [[workflow]], [[skill]], [[decision]] cần tạo/sửa. Ghi rõ trạng thái: đã làm / chờ distill.]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

#### Ví dụ

```markdown
---
type: lesson
title: "Estimate thiếu buffer kiểm thử hồi quy khi đụng module dùng chung"
tags: [estimate, quan-ly-du-an, qa]
related: [quy-trinh-estimate]
created: 2026-06-10
updated: 2026-06-10
confidence: low
status: active
---

## Bối cảnh

Dự án A, giai đoạn bàn giao tháng 5. Team 4 người, lần đầu sửa sâu vào module thanh toán dùng chung với 2 hệ thống khác.

## Kỳ vọng vs Thực tế

Kỳ vọng bàn giao 15/5 theo estimate 6 tuần. Thực tế bàn giao 27/5, trễ 12 ngày. Toàn bộ phần trễ nằm ở giai đoạn kiểm thử: phát sinh 23 lỗi hồi quy ở 2 hệ thống dùng chung module.

## Nguyên nhân gốc

Trễ vì lỗi hồi quy → vì estimate không có thời gian kiểm thử hồi quy cho hệ thống ngoài phạm vi dự án → vì template estimate chỉ tính công việc trong phạm vi, không có mục "ảnh hưởng chéo" → gốc: quy trình estimate coi mỗi dự án là một ốc đảo. Điều kiện làm trầm trọng: khách đổi phạm vi giữa chừng nhưng không ai re-estimate.

## Bài học

Đụng vào module dùng chung thì chi phí kiểm thử nằm ở các hệ thống xung quanh, không nằm trong dự án, và estimate phải tính phần đó.

## Khi áp dụng / Khi KHÔNG áp dụng

- Áp dụng: mọi dự án sửa module có từ 2 hệ thống sử dụng trở lên.
- KHÔNG áp dụng: module độc lập, chỉ một nơi dùng. Thêm buffer khi đó là lãng phí.

## Hành động hệ thống

- Sửa [[quy-trinh-estimate]]: thêm bước "liệt kê hệ thống dùng chung + buffer hồi quy" (chờ distill).
- Tạo decision "đổi phạm vi thì buộc re-estimate" (chờ distill).

## Changelog

- 2026-06-10: Tạo mới từ inbox
```

### 2.6. Project (tuỳ chọn, chỉ seed khi init xác nhận kho theo dõi theo dự án)

File: `wiki/project-slug.md`

Khác các type còn lại một bậc: `project` là trang **thực thể** (entity hub), không phải một mẩu tri thức. Vai trò: điểm neo để decision/lesson/pattern cùng dự án link về, trả lời câu "dự án X dạy ta điều gì". Type này KHÔNG nằm trong schema mặc định; `knowhow-init` chỉ thêm khi user xác nhận kho làm việc theo dự án/khách hàng/chiến dịch, hoặc schema evolution thêm sau khi đủ tín hiệu.

#### Frontmatter

```yaml
---
type: project
title: "Tên dự án"
tags: []
related: []
phase: planning | active | done
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: low | medium | high
status: active   # active | deprecated | archived
---
```

`phase` là trạng thái của DỰ ÁN (chạy/xong), tách khỏi `status` là vòng đời của TRANG. Dự án xong thì `phase: done` nhưng trang vẫn `status: active` vì tri thức còn được tra.

#### Body

```markdown
## Mục tiêu

[Dự án định đạt gì, đo bằng gì.]

## Phạm vi và mốc chính

[Phạm vi, mốc thời gian quan trọng, ai tham gia.]

## Kết quả so mục tiêu

[Điền khi dự án xong. Đạt/lệch ở đâu, số cụ thể. Lệch lớn thì trỏ [[lesson]] tương ứng.]

## Tri thức sinh ra từ dự án

[Danh sách [[decision-...]], [[lesson-...]], [[pattern-...]]. distill bồi thêm mỗi khi đúc kết item thuộc dự án này.]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

---

## 3. Skill Page Format

File: `skills/slug.md`

### Frontmatter

```yaml
---
type: skill
title: "Tên skill"
tags: []
trigger: "Khi nào nên dùng skill này"
input: "Mô tả đầu vào"
output: "Mô tả đầu ra"
reusable_across: []
promoted_from: ""   # TUỲ CHỌN. Nếu skill được nâng từ một wiki page, trỏ về: [[<slug-page-nguồn>]].
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
status: active   # active | deprecated | archived
---
```

### Body

```markdown
## Mục đích

[Skill này giải quyết vấn đề gì? Khi nào dùng?]

## Input / Output

**Input:** [Chi tiết đầu vào, format, constraints]

**Output:** [Chi tiết đầu ra, format]

## Các bước thực hiện

1. [Bước 1: chi tiết mức thao tác]
2. [Bước 2: ...]
3. ...

## Edge cases

- [Case đặc biệt 1 và cách xử lý]
- [Case đặc biệt 2 và cách xử lý]

## Changelog

- YYYY-MM-DD: Tạo mới, version 1.0
```

### Ví dụ

```markdown
---
type: skill
title: "Parse PDF hóa đơn thành structured JSON"
tags: [pdf, parsing, invoice]
trigger: "Khi có hoá đơn PDF (VAT Việt Nam) cần trích dữ liệu thành JSON"
input: "File PDF hóa đơn (VAT Việt Nam)"
output: "JSON object chứa seller, buyer, items, totals"
reusable_across: [accounting-module, report-generator]
created: 2026-06-05
updated: 2026-06-05
version: "1.0"
status: active
---

## Mục đích

Trích xuất thông tin từ hóa đơn PDF (VAT Việt Nam) thành dạng structured data. Dùng khi cần import hóa đơn tự động vào hệ thống kế toán.

## Input / Output

**Input:** File PDF hóa đơn VAT. Hỗ trợ cả scan (OCR) và electronic PDF.

**Output:** JSON object:
```json
{
  "invoice_number": "0012345",
  "seller": { "name": "...", "tax_id": "..." },
  "buyer": { "name": "...", "tax_id": "..." },
  "items": [{ "name": "...", "quantity": 1, "unit_price": 100000, "total": 100000 }],
  "subtotal": 100000,
  "vat_rate": 0.1,
  "vat_amount": 10000,
  "total": 110000
}
```

## Các bước thực hiện

1. Detect loại PDF: electronic (có text layer) hay scan (cần OCR).
2. Nếu scan: chạy OCR với Tesseract, lang=vie.
3. Extract text, dùng regex pattern cho format hóa đơn VAT VN.
4. Parse seller/buyer block: tìm "Đơn vị bán hàng", "Đơn vị mua hàng".
5. Parse items table: detect header row, iterate rows.
6. Parse totals: tìm "Cộng tiền hàng", "Thuế GTGT", "Tổng thanh toán".
7. Validate: kiểm tra subtotal + vat = total.

## Edge cases

- PDF scan mờ: OCR confidence < 80% thì flag để review thủ công.
- Hóa đơn nhiều trang: concat text trước khi parse.
- Format hóa đơn không chuẩn: trả về partial result với field `_warnings`.

## Changelog

- 2026-06-05: Tạo mới, version 1.0
```

---

## 4. Workflow Page Format

File: `workflows/slug.md`

### Frontmatter

```yaml
---
type: workflow
title: "Tên workflow"
tags: []
skills_used: []
trigger: "Khi nào kích hoạt"
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
status: active   # active | deprecated | archived
---
```

### Body

```markdown
## Khi nào dùng

[Điều kiện kích hoạt workflow. Ai kích hoạt? Tần suất?]

## Các bước

1. [Bước 1]
   → Dùng skill: [[skill-name]]
2. [Bước 2]
3. [Bước 3: điều kiện rẽ nhánh]
   - Nếu A: [làm X]
   - Nếu B: [làm Y]
4. ...

## Điều kiện rẽ nhánh

[Chi tiết các điểm rẽ nhánh, điều kiện if/else]

## Changelog

- YYYY-MM-DD: Tạo mới, version 1.0
```

Quy tắc trace: bước nào sinh ra từ một bài học thì ghi kèm nguồn ngay sau bước, dạng `(vì [[lesson-slug]])`. Người làm theo hiểu tại sao bước tồn tại, và khi bài học bị deprecated thì biết bước nào cần xét lại.

### Ví dụ

```markdown
---
type: workflow
title: "Quy trình xuất bản một bài blog"
tags: [content, blog, bien-tap, van-hanh]
skills_used: [viet-caption-theo-gu-thuong-hieu, toi-uu-seo-tieu-de, kiem-tra-chinh-ta]
trigger: "Khi có bản nháp bài blog cần đưa lên website"
created: 2026-06-06
updated: 2026-06-06
version: "1.0"
status: active
---

## Khi nào dùng

Khi cây bút đã có bản nháp hoàn chỉnh và bài cần được biên tập, tối ưu rồi xuất bản lên website. Thường chạy 2-3 lần mỗi tuần theo lịch nội dung.

## Các bước

1. Đọc soát bản nháp: cấu trúc, độ dài, có đúng đối tượng đọc không.
2. Biên tập câu chữ và kiểm tra chính tả.
   → Dùng skill: [[kiem-tra-chinh-ta]]
3. Tối ưu tiêu đề và phần mô tả cho SEO.
   → Dùng skill: [[toi-uu-seo-tieu-de]]
4. Chuẩn bị ảnh bìa và ảnh minh họa (đúng kích thước, có alt text).
5. Kiểm tra trước khi đăng:
   - Nếu bài có yếu tố pháp lý/số liệu: gửi người phụ trách duyệt, chờ OK rồi tiếp.
   - Nếu là bài thường: tiếp bước 6.
6. Lên lịch hoặc xuất bản trên website, gắn đúng chuyên mục và tag.
7. Soạn caption chia sẻ lên mạng xã hội.
   → Dùng skill: [[viet-caption-theo-gu-thuong-hieu]]
8. Cập nhật lịch nội dung: đánh dấu bài đã xuất bản, ghi link.

## Điều kiện rẽ nhánh

- **Bài có số liệu/pháp lý (bước 5)**: BẮT BUỘC qua người duyệt, không tự đăng.
- **Phát hiện trùng nội dung với bài cũ (bước 1)**: dừng lại, quyết định gộp/cập nhật bài cũ thay vì đăng bài mới.

## Changelog

- 2026-06-06: Tạo mới, version 1.0
```

---

## 5. Changelog Format

Mọi page (trừ inbox) đều có phần Changelog ở cuối.

```markdown
## Changelog

- YYYY-MM-DD: Mô tả thay đổi
```

Quy tắc:
- Mỗi lần cập nhật page, thêm 1 dòng mới lên đầu danh sách (mới nhất trước).
- Mô tả ngắn gọn, đủ hiểu thay đổi gì.
- Entry đầu tiên luôn là "Tạo mới từ inbox" (với wiki) hoặc "Tạo mới, version X.Y" (với skill/workflow).

---

## 6. Registry Formats

### 6.1. Skill Registry

File: `skills/registry.md`

```markdown
# Skill Registry

| Skill | Mô tả | Khi nào dùng | Version | Tags | Cập nhật |
|-------|--------|--------------|---------|------|----------|
| [[slug]] | mô tả ngắn | khi nào nên dùng (= trigger) | 1.0 | tag1, tag2 | YYYY-MM-DD |
```

Quy tắc:
- Mỗi skill có đúng 1 dòng.
- Sort theo alphabet.
- Cập nhật registry mỗi khi thêm/sửa/xóa skill.

### 6.2. Workflow Registry

File: `workflows/registry.md`

```markdown
# Workflow Registry

| Workflow | Mô tả | Khi nào dùng | Skills dùng | Version | Tags | Cập nhật |
|----------|--------|--------------|-------------|---------|------|----------|
| [[slug]] | mô tả ngắn | khi nào nên dùng (= trigger) | skill1, skill2 | 1.0 | tag1, tag2 | YYYY-MM-DD |
```

Quy tắc:
- Mỗi workflow có đúng 1 dòng.
- Sort theo alphabet.
- Cột "Khi nào dùng" lấy từ field `trigger` của workflow (tín hiệu để `knowhow-run` match task sang workflow, đối xứng với skill registry).
- Cột "Skills dùng" liệt kê các skill reference.
- Cập nhật registry mỗi khi thêm/sửa/xóa workflow.
