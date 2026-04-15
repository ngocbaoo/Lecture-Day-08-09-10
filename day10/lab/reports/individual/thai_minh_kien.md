# Báo Cáo Cá Nhân — Lab Day 10: Data Pipeline & Observability

**Họ và tên:** Thái Minh Kiên
**Vai trò:** Monitoring / Docs Owner
**Ngày nộp:** 15/04/2026

---

> **Tôi** phụ trách chính việc xây dựng hệ thống tài liệu (Documentation) và triển khai giám sát độ tươi mới của dữ liệu (Freshness Monitoring). Tôi đã hoàn thiện các file kiến trúc, contract và runbook, đồng thời đảm bảo các báo cáo chất lượng khớp với kết quả thực tế từ bộ phận Eval.

---

## 1. Tôi phụ trách phần nào? (80–120 từ)

**File / module:**
- `docs/pipeline_architecture.md`: Thiết kế sơ đồ luồng dữ liệu (Mermaid) và định nghĩa ranh giới trách nhiệm.
- `docs/data_contract.md`: Xây dựng schema chuẩn (cleaned) và các quy tắc cách ly dữ liệu (Quarantine).
- `docs/runbook.md`: Soạn thảo hướng dẫn xử lý sự cố khi hệ thống gặp lỗi chất lượng hoặc freshness.
- `monitoring/freshness_check.py`: Triển khai logic kiểm tra SLA 24h dựa trên manifest của pipeline.
- `reports/group_report.md`: Tổng hợp kết quả từ các thành viên để hoàn thiện báo cáo nhóm.

**Kết nối với thành viên khác:**
Tôi làm việc chặt chẽ với **NX Hai (Embed/Eval Owner)** để lấy các thông số `embed_upsert count` và kết quả từ file `after.csv` nhằm chứng minh tính Idempotent và chất lượng retrieval.

**Bằng chứng (commit / comment trong code):**
Trong file `monitoring/freshness_check.py`, tôi đã xử lý logic so sánh `age_hours` với `sla_hours` (dòng 51-59). Kết quả chạy thực tế với `run_id=2026-04-15T08-41Z` đã báo **FAIL** (120.7h) đúng như dự kiến.

---

## 2. Một quyết định kỹ thuật (100–150 từ)

Tôi đã quyết định đặt **SLA Freshness là 24 giờ**. Quyết định này dựa trên yêu cầu nghiệp vụ của CS + IT Helpdesk, nơi các chính sách thường không thay đổi theo từng phút nhưng cần cập nhật trong ngày. Trong quá trình giám sát bản chạy `2026-04-15T08-41Z`, hệ thống đã báo FAIL do dữ liệu nguồn từ ngày 10/04. Quyết định kỹ thuật này giúp nhóm nhận diện được sự chậm trễ của hệ thống nguồn (Batch lag), từ đó đưa ra cảnh báo kịp thời trong **Runbook** để đội vận hành có phương án xử lý (ví dụ: manually trigger export).

---

## 3. Một lỗi hoặc anomaly đã xử lý (100–150 từ)

Tôi đã xử lý vấn đề **Data Drift** trong chính sách hoàn tiền.
- **Triệu chứng:** Agent truy xuất nhầm thông tin 14 ngày làm việc.
- **Phát hiện:** `expectation[refund_no_stale_14d_window]` báo FAIL trong log.
- **Xử lý:** Tôi đã phối hợp tài liệu hóa quy trình này trong Runbook, hướng dẫn cách sử dụng `artifacts/quarantine/` để truy vết. Sau khi Cleaning Owner kích hoạt rule sửa lỗi, tôi xác nhận lại qua báo cáo Eval của bộ phận Embed: `contains_expected` đạt "yes" và `q_refund_window` trả về chính xác 7 ngày.

---

## 4. Bằng chứng trước / sau (80–120 từ)

Dựa trên kết quả chạy pipeline:
- **Run ID:** `2026-04-15T08-41Z`.
- **Evidence:** Cột `contains_expected` trong file `after.csv` đạt 4/4 "yes".
- **Refund Query:** Trả về "7 ngày làm việc" (Đã fix).
- **Freshness:** `FAIL {"age_hours": 120.701, "sla_hours": 24.0}`.

Bằng chứng này khẳng định mặc dù dữ liệu chưa "tươi" (stale), nhưng logic làm sạch và retrieval vẫn hoạt động cực kỳ chính xác.

---

## 5. Cải tiến tiếp theo (40–80 từ)

Tôi sẽ nâng cấp hệ thống giám sát để tự động gửi thông báo qua Slack khi Freshness check báo FAIL. Việc này giúp giảm thời gian phát hiện (MTTD) từ vài giờ (do phải check manual) xuống còn vài phút, đảm bảo đội vận hành can thiệp ngay khi dữ liệu nguồn bị trễ.
