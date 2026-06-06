---
name: knowhow-capture
description: "Ghi nhận knowhow từ cuộc trao đổi hoặc nguồn ngoài vào inbox dự án. Quét cuộc hội thoại, nhận diện quyết định, pattern, troubleshooting, ý tưởng skill/workflow, trình bày cho user duyệt rồi ghi vào .knowhow/inbox/. Trigger: 'knowhow capture', 'capture lại', 'ghi nhận knowhow', 'lưu bài học', khi user nói 'cái này hay, ghi lại đi', hoặc sau phiên làm việc muốn đúc kết."
---

# Knowhow Capture

Ghi nhận knowhow vào inbox dự án. Capture CHỈ ghi vào inbox, KHÔNG ghi thẳng vào wiki/skills/workflows.

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace hiện tại:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user:

> Chưa có `.knowhow/` trong workspace. Chạy `knowhow-init` trước để khởi tạo cấu trúc thư mục.

## Chế độ capture

### 1. Từ cuộc trao đổi (mặc định)

Quét conversation history hiện tại. Đọc transcript từ log nếu cần.

### 2. Từ nguồn ngoài

User cung cấp file (transcript, ghi chú, tài liệu). Đọc file, áp dụng cùng tiêu chí nhận diện.

Nếu user không chỉ rõ chế độ, hỏi:

> Capture từ cuộc trao đổi hiện tại hay từ file/nguồn ngoài?

## Tiêu chí nhận diện knowhow

| Loại | Tín hiệu nhận biết | Ví dụ |
|------|---------------------|-------|
| decision | "quyết định", "chọn X thay Y", "vì lý do..." | Chuyển từ REST sang GraphQL |
| pattern | Cách giải quyết lặp lại, "hay là dùng...", "trick này..." | Retry với jitter |
| troubleshooting | "lỗi", "fix", "nguyên nhân", "root cause" | Memory leak do không close connection |
| concept | Thuật ngữ mới, định nghĩa, giải thích khái niệm | "Bounded context" trong kiến trúc dự án |
| candidate-skill | Thao tác cụ thể, làm-theo-được, tái dùng cho task tương tự | Parse PDF hóa đơn; nâng từ một pattern page bị làm-theo lặp |
| candidate-workflow | Chuỗi bước, quy trình, checklist | Release flow |

**Lưu ý**: Một đoạn hội thoại có thể chứa nhiều loại knowhow. Tách riêng từng item.

**Ghi chú promote**: khi user (hoặc `knowhow-run`) bảo "việc này lặp lại, nâng [[<slug>]] thành skill", tạo một candidate-skill item, thêm `promote_of: <slug-page-nguồn>` trong frontmatter và trích nội dung page nguồn vào phần Chi tiết. distill sẽ ưu tiên đọc page nguồn rồi đề xuất tạo skill giữ liên kết ngược.

## Flow capture

### Bước 1: Quét nguồn

- Conversation: đọc toàn bộ history, lọc phần có tín hiệu knowhow.
- File: đọc nội dung, áp dụng cùng tiêu chí.

### Bước 2: Trình bày danh sách candidates

Hiển thị cho user, mỗi item gồm emoji + loại + tóm tắt 1-2 câu:

```
1. 📋 decision: Chọn PostgreSQL thay MongoDB vì cần transaction ACID
2. 🔧 pattern: Retry với exponential backoff + jitter cho external API
3. 🐛 troubleshooting: Memory leak do connection pool không release
4. 📖 concept: Bounded context là ranh giới logic trong domain model
5. ⚡ candidate-skill: Parse PDF hóa đơn thành structured JSON
6. 🔄 candidate-workflow: Release flow gồm 5 bước từ branch đến deploy
```

Emoji mapping:
- 📋 decision
- 🔧 pattern
- 🐛 troubleshooting
- 📖 concept
- ⚡ candidate-skill
- 🔄 candidate-workflow

### Bước 3: User duyệt

Hỏi user: giữ / sửa / bỏ từng item (hoặc batch). Cho phép user thêm item mà agent bỏ sót.

### Bước 4: Ghi vào inbox

Với mỗi item được duyệt:

1. Tạo file `inbox/YYYY-MM-DD-slug.md` theo format trong [page-formats.md](references/page-formats.md).
   - `slug`: tóm tắt 2-4 từ, kebab-case (ví dụ: `chon-postgresql`, `retry-jitter-pattern`).
   - Nếu trùng slug trong ngày, thêm suffix số: `-2`, `-3`.
2. **Luôn lưu nguồn vào `raw/`** (cả khi nguồn là hội thoại, không có file ngoài):
   - Ghi trích đoạn nguyên văn nguồn vào `raw/YYYY-MM-DD-slug.md` (cùng slug với inbox item).
   - Set frontmatter inbox `source_file: raw/YYYY-MM-DD-slug.md` (đường dẫn tương đối từ `.knowhow/`). KHÔNG để trống. KHÔNG trỏ vào chính file inbox.

### Bước 5: Ghi log

Thêm entry vào `wiki/log.md`:

```
## [YYYY-MM-DD] capture | Ghi nhận N items vào inbox từ [nguồn]
```

Trong đó `[nguồn]` = "conversation" hoặc tên file gốc.

### Bước 6: Xác nhận

Báo user kết quả: bao nhiêu item đã ghi, đường dẫn file.

Gợi ý chạy `knowhow-distill` để đúc kết inbox thành wiki/skill/workflow.

**Chế độ liền mạch (user solo)**: Nếu user gọi capture kèm cờ `--then-distill`, sau khi ghi inbox xong, chạy luôn `knowhow-distill` trong cùng phiên để duyệt + đúc kết liền 2 bước, không phải gọi lại.

## Quy tắc cứng

1. Capture CHỈ ghi vào `inbox/`. KHÔNG ghi thẳng vào `wiki/`, `skills/`, `workflows/`.
2. Đúc kết là việc của `knowhow-distill`. Capture không merge, không deduplicate.
3. Luôn trình bày candidates cho user duyệt trước khi ghi. KHÔNG tự động ghi.
4. Giữ nguyên ngữ cảnh gốc trong phần "Context gốc" để trace về sau.
5. Đọc format từ [page-formats.md](references/page-formats.md), không hardcode format.
