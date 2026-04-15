# Báo Cáo Cá Nhân — Lab Day 10: Data Pipeline & Observability

**Họ và tên:** Lê Minh Hoàng  
**Vai trò:** Ingestion Owner  
**Ngày nộp:** 2026-04-15  
**Độ dài:** 420 từ

---

## 1. Tôi phụ trách phần nào? (80–120 từ)

**File / module:**

- `etl_pipeline.py` — entrypoint ingestion, log run_id và raw_records
- `transform/cleaning_rules.py` — hàm `load_raw_csv()` để đọc CSV thô và xác định schema ingest
- `contracts/data_contract.yaml` / `docs/data_contract.md` — định nghĩa schema cleaned và nguồn dữ liệu canonical

**Mô tả trách nhiệm:**

Tôi chịu trách nhiệm phần ingestion: đọc file raw CSV (`data/raw/policy_export_dirty.csv`), xác định schema cần ingest, tạo `run_id`, và ghi log số lượng bản ghi đầu vào. Tôi đảm bảo rằng pipeline bắt đầu với dữ liệu thô đúng format và thông tin `run_id` được gắn xuyên suốt các artifact.

**Bằng chứng (code):**

```python
rows = load_raw_csv(raw_path)
raw_count = len(rows)
log(f"run_id={run_id}")
log(f"raw_records={raw_count}")
```

---

## 2. Một quyết định kỹ thuật (100–150 từ)

Tôi quyết định giữ phần ingest trong `etl_pipeline.py` đơn giản và chỉ tập trung vào việc đọc raw CSV, xác định schema và ghi log. Thay vì mở rộng trực tiếp các rule clean/quality ở bước này, tôi giữ `load_raw_csv()` chỉ chịu trách nhiệm load dữ liệu thô và `run_id`/`raw_records` được ghi sớm nhất có thể. Quyết định này giúp khi có lỗi ở bước sau, nhóm vẫn có thể kiểm tra ngay đầu vào bằng `artifacts/logs/run_<run_id>.log` mà không phải phân tích toàn bộ flow.

---

## 3. Một lỗi hoặc anomaly đã xử lý (100–150 từ)

Triệu chứng tôi gặp là cần xác nhận ingest đã giữ đúng dữ liệu đầu vào trước khi chuyển sang bước clean. Metric phát hiện là `raw_records` trong log. Tôi đã xử lý bằng cách thêm đoạn log `raw_records=10` ngay sau `load_raw_csv()` và lưu lại trong `artifacts/logs/run_2026-04-15T08-12Z.log`, giúp chứng minh pipeline đã ingest đúng 10 bản ghi ban đầu.

---

## 4. Bằng chứng trước / sau (80–120 từ)

**run_id:** `2026-04-15T08-12Z`  

Tôi đã kiểm tra log tại `artifacts/logs/run_2026-04-15T08-12Z.log` và thấy:

```text
run_id=2026-04-15T08-12Z
raw_records=10
```

Đây là bằng chứng trực tiếp cho việc ingest thành công 10 dòng raw CSV. Nếu người khác làm bước clean/quality tiếp theo, họ có thể dùng log này để đối chiếu dữ liệu đầu vào.

---

## 5. Cải tiến tiếp theo (40–80 từ)

Nếu có thêm 2 giờ, tôi sẽ tách rõ phần ingest riêng khỏi phần clean/validate bằng cách tạo module `ingest.py` chỉ chịu trách nhiệm load CSV và ghi `run_id`/`raw_records`, đồng thời thêm một bước sanity check đầu vào trước khi chuyển cho transform.

