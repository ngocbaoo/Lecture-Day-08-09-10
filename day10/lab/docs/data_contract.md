# Data contract — Lab Day 10

> Bắt đầu từ `contracts/data_contract.yaml` — mở rộng và đồng bộ file này.

---

## 1. Nguồn dữ liệu (source map)

| Nguồn | Phương thức ingest | Failure mode chính | Metric / alert |
|-------|-------------------|-------------------|----------------|
| `policy_export_dirty.csv` | Local CSV loading | Sai tên cột, thiếu dữ liệu bắt buộc | `raw_records`, `quarantine_count` |
| `it_helpdesk_faq` | Manual entry / Export | Mojibake encoding | `text_repaired_count` |

---

## 2. Schema cleaned

| Cột | Kiểu | Bắt buộc | Ghi chú |
|-----|------|----------|---------|
| chunk_id | string | Có | Hash SHA-256 (doc_id + content + seq) |
| doc_id | string | Có | Phải thuộc ALLOWED_DOC_IDS |
| chunk_text | string | Có | Minimum 8 chars, repaired mojibake |
| effective_date | date | Có | ISO-8601 YYYY-MM-DD |
| exported_at | datetime | Có | ISO-8601 UTC |

---

## 3. Quy tắc quarantine vs drop

- **Quarantine:** Record không qua được cleaning rules (unknown doc_id, ngày sai định dạng, ngày ở tương lai, duplicate) được đẩy vào `artifacts/quarantine/`.
- **Drop:** Chỉ thực hiện drop nếu dữ liệu hoàn toàn không có `doc_id` hoặc rỗng tất cả các trường quan trọng (không thể quarantine vì không có context).
- **Approval:** Data Owner kiểm tra file quarantine, sửa source export và rerun pipeline.

---

## 4. Phiên bản & canonical

- **Source of truth:** Các file text trong `data/docs/` (kế thừa từ Day 08/09).
- **Policy refund:** Canonical version là **v4** với quy định hoàn tiền trong **7 ngày làm việc**. Mọi bản export ghi 14 ngày đều được coi là stale và bị pipeline tự động sửa (fix-on-ingest).
- **HR Policy:** Chỉ chấp nhận các bản policy có hiệu lực từ **2026-01-01** trở đi.
