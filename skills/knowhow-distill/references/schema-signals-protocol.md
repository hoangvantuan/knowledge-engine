# Giao thức phát tín hiệu schema-signals

Một nơi định nghĩa cách phát tín hiệu vào `.knowhow/schema-signals.md`. Dùng chung cho `knowhow-distill`, `knowhow-query`, `knowhow-run` (cùng cơ chế ghi, chỉ khác nội dung dòng). Đổi format thì sửa ở đây, không sửa rải rác.

## Bốn loại tín hiệu

| Loại | Ai phát | Khi nào emit (điều kiện ở SKILL.md skill phát) |
|---|---|---|
| `no-fit-type` | distill | Item là loại tri thức không nằm trong wiki type của SCHEMA, buộc xếp tạm |
| `adhoc-section` | distill | Phải thêm section ngoài template chuẩn, VÀ section đó đã lặp ở ≥1 page khác cùng type |
| `query-miss` | query | Phải chắp ≥3 page mới trả lời được, hoặc không có page sạch trả lời trực tiếp |
| `promote-candidate` | query, run | Một wiki page pattern/troubleshooting bị dùng lặp để LÀM THEO (ứng viên nâng thành skill) |

## Format dòng tín hiệu

```
- [YYYY-MM-DD] <nguồn> | <loại> | <chi tiết ngắn> | related: <slug-hoặc-tag:chủ-đề>
```

- `<nguồn>` ∈ distill | query | run
- `<loại>` ∈ no-fit-type | adhoc-section | query-miss | promote-candidate
- `related`: slug page nguồn (cho `promote-candidate`) hoặc `tag:<chủ-đề>` (cho loại gom theo cụm)

## Cách ghi (chèn dưới "Đang chờ xử lý", KHÔNG append cuối file)

Cuối file là vùng `## Đã xử lý`, nên phải chèn NGAY DƯỚI heading `## Đang chờ xử lý`. THAY `YYYY-MM-DD` bằng ngày thật, slug/tag thật TRƯỚC khi chạy:

```bash
awk '/^## Đang chờ xử lý$/{print; print "- [YYYY-MM-DD] <nguồn> | <loại> | <chi tiết> | related: <...>"; next} 1' \
  .knowhow/schema-signals.md > /tmp/ss && mv /tmp/ss .knowhow/schema-signals.md
```

Nếu `.knowhow/schema-signals.md` thiếu, tạo lại theo template `knowhow-init` (tiêu đề `# Schema Signals` + hai heading `## Đang chờ xử lý`, `## Đã xử lý`) rồi mới ghi.

## Quy tắc chung

- **Một phiếu mỗi lần, KHÔNG dedupe.** Mỗi lần điều kiện lặp lại là một dòng mới cùng slug. distill/schema-review đếm số dòng để áp ngưỡng.
- **Đã promote thì thôi.** Page đã có skill tương ứng (`grep -F 'promoted_from: [[<slug>]]' .knowhow/skills` ra kết quả) thì KHÔNG emit `promote-candidate` nữa.
- **Tín hiệu không chặn luồng chính.** Vẫn xử lý item/câu hỏi bình thường; tín hiệu chỉ là ghi chú cho `schema-review`/`distill` sau.
- **Phân vai tổng hợp.** `promote-candidate` do `knowhow-distill` tổng hợp (sinh artifact actionable, ngưỡng ≥3 phiếu cùng slug). `knowhow-lint schema-review` BỎ QUA dòng `promote-candidate`, chỉ xử lý no-fit-type/adhoc-section/query-miss.
