# Báo Cáo Nhóm — Lab Day 10: Data Pipeline & Data Observability

**Tên nhóm:** D1
**Thành viên:**
| Tên | Vai trò (Day 10) | Email |
|-----|------------------|-------|
| Lê Minh Hoàng | Ingestion / Raw Owner | phamhaihau1976@gmail.com |
| Tạ Bảo Ngọc | Cleaning & Quality Owner | ngoctabao@gmail.com |
| Nguyễn Xuân Hải | Embed & Idempotency Owner | nxhaicr7@gmail.com |
| Thái Minh Kiên | Monitoring / Docs Owner | minhkien242003@gmail.com |

**Ngày nộp:** 15/04/2026
**Repo:** https://github.com/ngocbaoo/Lecture-Day-08-09-10
**Độ dài khuyến nghị:** 600–1000 từ

---

> **Nộp tại:** `reports/group_report.md`  
> **Deadline commit:** xem `SCORING.md`.  
> Phải có **run_id**, **đường dẫn artifact**, và **bằng chứng before/after**.

---

## 1. Pipeline tổng quan (150–200 từ)

Hệ thống pipeline dữ liệu Day 10 tập trung vào việc xử lý các lỗi dữ liệu thô (Dirty CSV) để đảm bảo Agent không trả về các thông tin cũ hoặc sai lệch. Quy trình bao gồm: Ingest từ `policy_export_dirty.csv`, chuẩn hóa qua `cleaning_rules.py`, kiểm soát chất lượng qua `expectations.py` và nạp vào ChromaDB một cách idempotent.

**Tóm tắt luồng:**
- **Ingest:** Tải 10 bản ghi thô.
- **Clean:** Loại bỏ duplicate và fix quy định refund 14 -> 7 ngày. Kết quả thu được 6 bản ghi sạch.
- **Validate:** Chạy 7 expectations. Toàn bộ PASSED trong bản chạy chuẩn.
- **Embed:** Upsert 6 bản ghi vào collection `day10_kb`.

**Lệnh chạy một dòng:**
```bash
python etl_pipeline.py run --run-id 2026-04-15T08-41Z
```

---

## 2. Cleaning & expectation (150–200 từ)

Nhóm đã triển khai các rule để xử lý dữ liệu thực tế:

### 2a. Bảng metric_impact

| Rule / Expectation mới (tên ngắn) | Trước (số liệu) | Sau / khi inject (số liệu) | Chứng cứ (log / CSV / commit) |
|-----------------------------------|------------------|-----------------------------|-------------------------------|
| `raw_records` | 10 | 10 | Console Log |
| `cleaned_records` | 6 | 6 | `artifacts/cleaned/` |
| `embed_upsert` (Idempotent) | 6 (lần 1) | 6 (lần 2) | Console Log |
| `q_refund_window` (Contains Expected) | No (Bad run) | Yes (Clean run) | `after.csv` |

**Rule chính:**
- **Auto-fix Refund:** Tự động sửa text "14 ngày" thành "7 ngày" kèm tag `[cleaned: stale_refund_window]`.
- **HR Stale Version:** Quarantine các policy cũ trước năm 2026.
- **Mojibake Fix:** Sửa lỗi hiển thị phông chữ tiếng Việt.

---

## 3. Before / after ảnh hưởng retrieval hoặc agent (200–250 từ)

Dựa trên kết quả từ bộ phận Eval/Embed:

**Kịch bản inject:**
Khi chạy lần 3 với flag `--no-refund-fix --skip-validate`, hệ thống nạp dữ liệu lỗi (14 ngày) vào database mà không cảnh báo.

**Kết quả định lượng (từ after.csv):**
- **Độ chính xác:** 100% `contains_expected` là "yes" cho 4 câu hỏi vàng.
- **Kết quả Refund:** Câu hỏi về thời hạn hoàn tiền trả về đúng **7 ngày**.
- **Kết quả HR:** Câu hỏi về ngày phép năm trả về đúng chính sách **2026** (12 ngày phép).
- **Hits Forbidden:** 100% "no", đảm bảo không lấy nhầm dữ liệu bẩn.

---

## 4. Freshness & monitoring (100–150 từ)

SLA được đặt là **24 giờ**. Trong manifest `2026-04-15T08-41Z`, kết quả báo **FAIL** vì `age_hours` lên tới 121h (do source data từ ngày 10/04). Điều này giúp nhóm nhận diện rõ sự lệch pha giữa hệ thống nguồn và pipeline.

---

## 5. Liên hệ Day 09 (50–100 từ)

Dữ liệu sạch sau pipeline giúp Agent của Day 09 trả về câu trả lời chính xác cho người dùng (ví dụ: không trả lời sai về 14 ngày hoàn tiền). Việc tách biệt tầng dữ liệu này giúp Agent tập trung vào logic hội thoại thay vì phải lo lắng về tính đúng đắn của tài liệu.

---

## 6. Rủi ro còn lại & việc chưa làm

- **Freshness SLA:** Cần cập nhật hệ thống nguồn để export đều đặn hơn nhằm đáp ứng SLA 24h.
- **Storage:** Vẫn cần theo dõi tình trạng Disk I/O của Chroma trên các môi trường khác nhau.
