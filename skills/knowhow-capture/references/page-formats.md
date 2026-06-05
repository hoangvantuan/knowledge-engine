# Page Formats: Knowhow System

Tài liệu tham chiếu chứa tất cả format page cho toàn bộ Knowhow System.
Dùng chung cho `knowhow-capture` (ghi inbox) và `knowhow-distill` (đúc kết wiki/skill/workflow).

---

## 1. Inbox Item Format

Mỗi item capture ghi vào `inbox/YYYY-MM-DD-slug.md`.

### Frontmatter

```yaml
---
type: decision | pattern | troubleshooting | concept | candidate-skill | candidate-workflow
title: "Tên ngắn gọn"
tags: []
captured_from: conversation | file-import
captured_at: "YYYY-MM-DDTHH:mm"
source_file: "raw/YYYY-MM-DD-slug.md"  # BẮT BUỘC. Trích đoạn nguồn (cả hội thoại) luôn lưu vào raw/. Không để trống.
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
title: "Chọn PostgreSQL thay MongoDB"
tags: [database, architecture]
related: [payment-module-design]
created: 2026-06-05
updated: 2026-06-05
confidence: high
---

## Bối cảnh

Module thanh toán yêu cầu transaction ACID xuyên nhiều bảng. Cần chọn database phù hợp.

## Các lựa chọn đã xét

| Lựa chọn | Ưu | Nhược |
|-----------|-----|-------|
| MongoDB | Schema linh hoạt, horizontal scale dễ | Multi-doc transaction chậm, latency +3x |
| PostgreSQL | ACID native, mature ecosystem | Schema cứng hơn, vertical scale trước |

## Quyết định

Dùng PostgreSQL làm database chính cho toàn bộ hệ thống.

## Lý do

Payment flow là core, cần data integrity tuyệt đối. Benchmark cho thấy MongoDB multi-doc transaction latency tăng 3x so với PostgreSQL transaction.

## Hệ quả

- Schema migration cần quản lý chặt chẽ hơn.
- Horizontal scale phức tạp hơn nếu cần sau này (xem xét Citus).
- Data integrity cho payment flow được đảm bảo.

## Changelog

- 2026-06-05: Tạo mới từ inbox
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
---
```

#### Body

```markdown
## Bối cảnh

[Khi nào áp dụng pattern này? Điều kiện tiên quyết?]

## Nội dung

[Cách làm chi tiết, từng bước nếu cần]

## Ví dụ

[Ví dụ từ dự án thực tế, có code nếu phù hợp]

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
---
```

#### Body

```markdown
## Định nghĩa

[Nghĩa của khái niệm trong bối cảnh dự án. Không phải định nghĩa sách giáo khoa chung chung.]

## Liên quan

[Mối quan hệ với các concept/decision/pattern khác]

## Ví dụ

[Ví dụ cụ thể trong dự án]

## Changelog

- YYYY-MM-DD: Tạo mới từ inbox
```

#### Ví dụ

```markdown
---
type: concept
title: "Bounded Context"
tags: [ddd, architecture]
related: [aggregate-root, ubiquitous-language]
created: 2026-06-05
updated: 2026-06-05
---

## Định nghĩa

Bounded Context là ranh giới logic trong đó một domain model cụ thể có hiệu lực. Trong dự án này, mỗi bounded context tương ứng với một module (payment, inventory, user).

Cùng một entity (ví dụ "User") có thể có nghĩa khác nhau giữa các context: Payment context quan tâm billing info, User context quan tâm profile.

## Liên quan

- [[aggregate-root]]: Mỗi bounded context có các aggregate root riêng.
- [[ubiquitous-language]]: Ngôn ngữ chung CHỈ có hiệu lực trong 1 bounded context.

## Ví dụ

Module Payment có `Customer` (chỉ giữ billing info). Module User có `User` (giữ profile, preferences). Hai entity khác nhau dù cùng đại diện "người dùng".

## Changelog

- 2026-06-05: Tạo mới từ inbox
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
title: "Memory leak do connection pool không release"
tags: [memory, database, production]
related: [connection-pooling-pattern]
created: 2026-06-05
updated: 2026-06-05
confidence: high
---

## Triệu chứng

- Memory usage tăng liên tục sau deploy, đạt OOM sau ~4 giờ.
- Số connection database tăng không giảm.
- Response time tăng dần.

## Nguyên nhân gốc

Code mới thêm query trong middleware nhưng dùng `pool.connect()` mà không gọi `client.release()` trong error path. Khi có exception, connection bị giữ vĩnh viễn.

## Cách fix

1. Wrap tất cả `pool.connect()` trong `try/finally` với `client.release()` ở `finally`.
2. Hoặc dùng `pool.query()` thay vì `pool.connect()` + `client.query()` cho single query.
3. Set `pool.max = 20` và `idleTimeoutMillis = 30000` để tự cleanup connection cũ.

## Phòng ngừa

- Lint rule: cấm `pool.connect()` trực tiếp, bắt buộc dùng wrapper function có release.
- Monitor: alert khi connection count > 80% pool size.

## Changelog

- 2026-06-05: Tạo mới từ inbox
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
input: "Mô tả đầu vào"
output: "Mô tả đầu ra"
reusable_across: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
version: "1.0"
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
input: "File PDF hóa đơn (VAT Việt Nam)"
output: "JSON object chứa seller, buyer, items, totals"
reusable_across: [accounting-module, report-generator]
created: 2026-06-05
updated: 2026-06-05
version: "1.0"
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

### Ví dụ

```markdown
---
type: workflow
title: "Release flow"
tags: [release, deployment, ci-cd]
skills_used: [run-test-suite, build-docker-image, deploy-to-k8s]
trigger: "Khi merge vào branch main và muốn release lên production"
created: 2026-06-05
updated: 2026-06-05
version: "1.0"
---

## Khi nào dùng

Sau khi feature branch đã merge vào main. Tech lead quyết định release. Thường 1-2 lần/tuần.

## Các bước

1. Tạo release branch `release/vX.Y.Z` từ main.
2. Chạy full test suite.
   → Dùng skill: [[run-test-suite]]
3. Kiểm tra kết quả test:
   - Nếu pass: tiếp bước 4.
   - Nếu fail: fix trên release branch, quay lại bước 2.
4. Build Docker image với tag version.
   → Dùng skill: [[build-docker-image]]
5. Deploy lên staging, smoke test 30 phút.
   → Dùng skill: [[deploy-to-k8s]]
6. Kiểm tra staging:
   - Nếu OK: deploy lên production.
   - Nếu có issue: rollback staging, fix, quay lại bước 2.
7. Tag release trên git, viết release notes.
8. Thông báo team qua Slack.

## Điều kiện rẽ nhánh

- **Test fail (bước 3)**: KHÔNG được skip. Fix xong mới tiếp.
- **Staging có issue (bước 6)**: Rollback ngay. Nếu issue nhỏ, fix trên release branch. Nếu issue lớn, abort release và tạo hotfix.

## Changelog

- 2026-06-05: Tạo mới, version 1.0
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

| Skill | Mô tả | Version | Tags | Cập nhật |
|-------|--------|---------|------|----------|
| [[slug]] | mô tả ngắn | 1.0 | tag1, tag2 | YYYY-MM-DD |
```

Quy tắc:
- Mỗi skill có đúng 1 dòng.
- Sort theo alphabet.
- Cập nhật registry mỗi khi thêm/sửa/xóa skill.

### 6.2. Workflow Registry

File: `workflows/registry.md`

```markdown
# Workflow Registry

| Workflow | Mô tả | Skills dùng | Version | Cập nhật |
|----------|--------|-------------|---------|----------|
| [[slug]] | mô tả ngắn | skill1, skill2 | 1.0 | YYYY-MM-DD |
```

Quy tắc:
- Mỗi workflow có đúng 1 dòng.
- Sort theo alphabet.
- Cột "Skills dùng" liệt kê các skill reference.
- Cập nhật registry mỗi khi thêm/sửa/xóa workflow.
