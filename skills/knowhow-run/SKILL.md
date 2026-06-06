---
name: knowhow-run
description: "Entrypoint chủ động để tiêu thụ skill/workflow đã đúc kết trong .knowhow/. Ba nhịp: tra registry → load file bó khớp → đọc hết rồi làm theo. Nhận 3 dạng input: tên bó cụ thể (load thẳng), mô tả task (tra registry theo trigger/Mô tả/tags), hoặc rỗng (liệt kê bó khả dụng). Workflow gặp skill con resolve đệ quy. KHÔNG sản xuất, KHÔNG ghi vào .knowhow/. Trigger: 'knowhow run', 'chạy skill X', 'làm theo workflow Y', 'dùng knowhow để làm', khi bắt đầu task thuộc domain dự án và cần dùng bó đã tích luỹ."
---

# Knowhow Run

Entrypoint chủ động để tra, load và làm theo skill/workflow đã đúc kết. Mục tiêu: thay vì liếc một dòng mô tả trong registry rồi tự làm theo trí nhớ, agent mở đúng file bó và làm theo nội dung thật.

Skill/workflow cùng là tri thức như wiki, khác duy nhất là chúng *hành động được*. Vì chỉ là tri thức actionable dạng markdown, tiêu thụ KHÔNG cần execution engine. Chỉ cần ba nhịp:

```
tra registry/index  →  load file bó khớp  →  đọc kỹ rồi làm theo
```

(bó = một skill hoặc một workflow đã đúc kết trong .knowhow/)

## Precondition

Kiểm tra `.knowhow/` tồn tại trong workspace:

```bash
ls -d .knowhow/ 2>/dev/null
```

Nếu không tồn tại, dừng và hướng dẫn user chạy `knowhow-init` trước.

## Phạm vi agent

`knowhow-run` thuần đọc-file-rồi-làm-theo, không thao tác đặc thù agent nào. Nó là skill vận hành, sống ở config agent, không nhúng vào sản phẩm `.knowhow/`.

## Phân vai với knowhow-query

|          | knowhow-query           | knowhow-run            |
| -------- | ----------------------- | ---------------------- |
| Nguồn    | wiki                    | skills + workflows     |
| Bản chất | tri thức để biết        | tri thức để làm        |
| Output   | câu trả lời + trích dẫn | việc được làm theo bó  |

Task "cần biết/tra cứu" → dùng `knowhow-query`. Task "cần làm theo một quy trình đã đúc kết" → dùng skill này.

## Ba dạng input

Xác định input thuộc dạng nào rồi rẽ nhánh:

| Dạng input | Ví dụ | Nhịp bắt đầu |
|---|---|---|
| Tên bó cụ thể | "chạy skill parse-invoice", "làm theo workflow release-flow" | Bỏ qua Tra, vào thẳng Load |
| Mô tả task | "tôi có file PDF hoá đơn cần trích dữ liệu" | Bắt đầu từ Tra |
| Rỗng | "knowhow run" (không kèm gì) | Liệt kê bó khả dụng cho user chọn |

## Ba nhịp

### Nhịp 1: Tra (chỉ khi input là mô tả task hoặc rỗng)

1. Đọc `.knowhow/skills/registry.md` và `.knowhow/workflows/registry.md`. (Đầu phiên agent đã đọc `SCHEMA.md` + index nên biết có bó nào.)
2. Nếu input RỖNG: liệt kê các bó khả dụng (tên + cột `Khi nào dùng` + `Mô tả`) cho user chọn, rồi dừng chờ chọn.
3. Nếu input là MÔ TẢ TASK: match task với các bó. Thứ tự ưu tiên khi match:
   - Cột `Khi nào dùng` (field `trigger`): tín hiệu mạnh nhất, mô tả đúng tình huống kích hoạt.
   - Cột `Mô tả`.
   - Cột `Tags`.
   Rút 3-5 từ khoá từ task, đối chiếu. Nếu registry chưa đủ để chắc, grep nội dung thật:
   ```bash
   grep -ril "<từ khoá>" .knowhow/skills .knowhow/workflows
   ```
4. Chọn bó khớp nhất:
   - Đúng 1 bó khớp rõ → sang Nhịp 2.
   - Nhiều bó khớp → trình bày danh sách ngắn (tên + `Khi nào dùng`), hỏi user chọn.
   - Không bó nào khớp → nói thẳng "chưa có skill/workflow cho việc này", gợi ý `knowhow-capture` + `knowhow-distill` để đúc kết sau. KHÔNG bịa một bó.

### Nhịp 2: Load

1. Mở đúng file bó đã chọn:
   - Skill: `.knowhow/skills/<slug>.md`
   - Workflow: `.knowhow/workflows/<slug>.md`
   - Nếu type đã tách subfolder (schema tiến hoá), tìm đệ quy:
     ```bash
     find .knowhow/skills .knowhow/workflows -name "<slug>.md"
     ```
2. Đọc HẾT nội dung file. KHÔNG quyết định dựa trên một dòng registry. Một dòng mô tả không đủ để làm đúng; phải đọc cả các bước, edge case, điều kiện rẽ nhánh.

### Nhịp 3: Làm theo

1. Thực hiện lần lượt các bước ghi trong file bó. Bám sát nội dung thật, không tự chế thêm bước.
2. Nếu là **workflow** và gặp `→ Dùng skill: [[X]]`:
   - Resolve `[[X]]` về file skill con (`.knowhow/skills/X.md`, hoặc `find` nếu subfolder).
   - Quay lại Nhịp 2 (Load) cho skill con: đọc hết rồi làm theo.
   - Đệ quy tự nhiên, KHÔNG cần orchestrator. Làm xong skill con thì quay lại bước kế của workflow.
3. Tôn trọng "Điều kiện rẽ nhánh" của workflow: theo đúng nhánh, không skip bước bị chặn.
4. Nếu bó tham chiếu wiki page `[[slug]]` để lấy bối cảnh, đọc page đó khi cần (như knowhow-query làm), nhưng trọng tâm vẫn là làm theo bó.

## Quy tắc cứng

1. **KHÔNG ghi gì vào `.knowhow/`.** run thuần tiêu thụ: không tạo inbox item, không sửa wiki/skill/workflow, KHÔNG ghi `wiki/log.md`.
2. **Đọc hết file bó trước khi làm.** Cấm liếc một dòng registry rồi bịa các bước.
3. **Không bịa bó.** Không có bó khớp thì nói không có, gợi ý đúc kết. Không tự nghĩ ra quy trình rồi gán cho một bó không tồn tại.
4. **Không sản xuất.** Nếu trong lúc làm phát hiện bó thiếu/sai bước, KHÔNG tự sửa bó ở đây. Ghi nhận và gợi ý user chạy `knowhow-distill` để refine. (Tách tiêu thụ khỏi sản xuất.)
5. **Tôn trọng cửa duy nhất.** Bài học mới sinh trong lúc chạy (nếu đáng lưu) đi qua `knowhow-capture` → inbox, KHÔNG ghi thẳng.

## Edge cases

- **Bó có `status: deprecated`**: vẫn load được nhưng cảnh báo "bó này deprecated, có thể có cách mới hơn", hỏi user có tiếp không.
- **Bó có `status: archived`**: nằm trong `archive/`, KHÔNG nên chạy. Báo user bó đã lỗi thời.
- **Workflow trỏ skill con không resolve** (`[[X]]` không tìm thấy file): dừng nhịp đó, báo link hỏng, gợi ý chạy `knowhow-lint quick` để bắt link hỏng.
- **Registry rỗng** (chưa có bó nào): báo "dự án chưa đúc kết skill/workflow nào", gợi ý `knowhow-capture` + `knowhow-distill`.
