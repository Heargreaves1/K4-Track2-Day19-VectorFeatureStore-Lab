# Reflection — Lab 19

**Tên:** Nguyễn Nguyên Phong
**Cohort:** A20-K4
**Path đã chạy:** lite (Windows 11, Python 3.12.6, embedding `bge-small-en-v1.5`)

---

## Câu hỏi (≤ 200 chữ)

> Trên golden set 50 queries, mode nào thắng ở loại query nào (`exact` /
> `paraphrase` / `mixed`), và tại sao? Khi nào bạn **không** dùng hybrid
> (i.e. khi nào pure BM25 hoặc pure vector là lựa chọn đúng)?

| Precision@10 | BM25 | Vector | Hybrid |
|---|---:|---:|---:|
| exact (15) | **96,7%** | 88,7% | **96,7%** |
| paraphrase (15) | **33,3%** | 24,0% | 32,0% |
| mixed (20) | 97,0% | 98,5% | **100%** |
| Trung bình | 77,8% | 73,2% | **78,6%** |

- **exact → BM25** (hybrid hoà): query là thuật ngữ có nguyên văn trong doc (`PostgreSQL replication sharding`); từ hiếm có IDF cao nên BM25 khớp chính xác, còn vector làm loãng tín hiệu từ khoá.
- **paraphrase → BM25, ngược kỳ vọng**: `bge-small-en` là model tiếng Anh, tokenizer bỏ dấu ("mở/mỡ/mợ" → `mo`) và cắt âm tiết thành subword, nên vector chỉ được 24%. Cần model đa ngữ (bge-m3) để vector thắng.
- **mixed → hybrid**: query trộn thuật ngữ và tiếng Việt nên cả hai retriever đều có tín hiệu; RRF cộng `1/(60+rank)`, doc được cả hai xếp cao nổi lên đầu.

**Không dùng hybrid khi:**
- *Pure BM25*: tra mã lỗi, SKU, số điều luật; chưa có embedding tốt cho ngôn ngữ/domain; cần latency thấp nhất (P99 keyword 3,9 ms so với hybrid 10,8 ms).
- *Pure vector*: query chủ yếu là câu tự nhiên hoặc đa ngữ, và có model đa ngữ mạnh.
- Một retriever quá yếu thì RRF kéo kết quả xuống: paraphrase hybrid 32,0% < BM25 33,3%.

---

## Điều ngạc nhiên nhất khi làm lab này

Ở NB6, toàn bộ khoảng thua của `agentic (+filter)` đến từ một lỗi substring: keyword `"ai"` khớp nhầm vào "f**ai**lure" và "h**ai** yếu tố", filter sai cụm vẫn trả đủ 8 doc nên bước phản tỉnh không hề phát hiện. Ở NB7, ngưỡng 0,75 trả lời sai 36% câu lẽ ra phải MISS — corpus này cần ~0,86.

---

## Ghi chú kỹ thuật (Windows + feast 0.66)

- **NB3:** gọi API qua `127.0.0.1` thay vì `localhost` — trên Windows `localhost` thử IPv6 trước, mỗi request mất thêm ~2 s.
- **NB8:** thêm `value_type=ValueType.STRING` cho entity `user` trong `app/feast_repo_ondemand/definitions.py` — thiếu nó, feast 0.66 suy `user_id` thành kiểu JSON và `materialize` lỗi.
- **NB4:** feast 0.66 bỏ hẳn entity row không có feature hợp lệ, nên PIT join được left-merge lại với `entity_df` để giữ đủ 3 dòng (`u_001` = NaN, đúng tinh thần PIT); thêm cell `feast feature-views list`.
- **NB6/NB7:** thêm cell phân tích trên số đo thật (vì sao `+filter` thua; chọn ngưỡng cache).

---

## Bonus challenge

- [ ] Đã làm bonus (xem `bonus/`)
- [ ] Pair work với: —
