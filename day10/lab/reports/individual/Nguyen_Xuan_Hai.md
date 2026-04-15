# Báo cáo cá nhân — mẫu GV (reference)

**Họ và tên:** Nguyễn Xuân Hải 
**Vai trò:** Cleaning & Quality  
**Độ dài:** ~450 từ (mẫu)

---

## 1. Phụ trách

Tôi chịu trách nhiệm toàn bộ giai đoạn cuối của Pipeline: từ việc thực thi các quy tắc làm sạch dữ liệu (Cleaning Rules), kiểm định chất lượng đầu ra (Expectations) đến việc đồng bộ hóa dữ liệu vào Vector Database.

Tôi trực tiếp triển khai logic trong transform/cleaning_rules.py để xử lý các lỗi dữ liệu về thời gian và nội dung rỗng, đồng thời quản lý file quality/expectations.py để đảm bảo dữ liệu trước khi Embed phải đạt chuẩn "Freshness". Tôi kết nối trực tiếp với công đoạn Retrieval thông qua việc xuất và đối chiếu các file manifest và after.csv.

Bằng chứng: Đã đẩy toàn bộ mã nguồn, dữ liệu làm sạch và thư mục artifacts/ lên nhánh cá nhân **nxhai** trên repo github của nhóm.
---

## 2. Quyết định kỹ thuật

**Cơ chế Idempotency & Pruning**: Đây là quyết định quan trọng nhất. Thay vì chỉ đơn giản là add thêm vector (gây phình database và làm nhiễu kết quả), tôi triển khai cơ chế Upsert + Prune. Nếu một dòng dữ liệu bị sửa (ví dụ: từ "14 ngày" thành "7 ngày"), hệ thống sẽ cập nhật thay vì tạo mới. Nếu một dòng bị xóa khỏi file gốc, hệ thống sẽ tự động thực hiện embed_prune_removed để dọn dẹp Vector DB. Điều này đảm bảo kết quả Top-1 luôn là duy nhất và chính xác nhất.

**Halt vs Warn (Xử lý Freshness)**: Đối với các chính sách quan trọng như SLA hay Refund, tôi áp dụng cơ chế Halt nếu dữ liệu đã quá 24h so với thời điểm xuất (exported_at). Điều này ngăn chặn việc hệ thống RAG trả lời dựa trên chính sách cũ đã hết hạn, đảm bảo tính an toàn cho vận hành.

---

## 3. Sự cố / anomaly

Trong quá trình thử nghiệm, tôi phát hiện dù file cleaned.csv đã được sửa nội dung chính sách hoàn tiền thành "7 ngày", nhưng kết quả Retrieval vẫn trả về "14 ngày" (hits_forbidden=true).

Nguyên nhân: Do thiếu cơ chế Prune, các vector cũ của chính sách lỗi vẫn tồn tại song song với vector mới trong ChromaDB.

Giải pháp: Tôi đã bổ sung logic so sánh set(prev_ids) - set(current_ids) trong etl_pipeline.py để triệt tiêu hoàn toàn các vector "mồ côi". Kết quả sau đó đạt hits_forbidden=no.

---

## 4. Before/after
Tôi đã thực hiện đánh giá định lượng thông qua file artifacts/eval/after.csv:

Trước khi xử lý: Câu hỏi q_refund_window trả về thông tin cũ (14 ngày), cột contains_expected báo FAIL.

Sau khi chạy Pipeline chuẩn: 
**Log**: embed_upsert count=6 (giữ nguyên qua nhiều lần chạy, chứng minh tính Idempotent).
(.venv) nxhai@nxhai:~/AI_thucchien/Lecture-Day-08-09-10/day10/lab$ python etl_pipeline.py run

run_id=2026-04-15T08-38Z

raw_records=10

cleaned_records=6

quarantine_records=4

cleaned_csv=artifacts/cleaned/cleaned_2026-04-15T08-38Z.csv

quarantine_csv=artifacts/quarantine/quarantine_2026-04-15T08-38Z.csv

expectation[min_one_row] OK (halt) :: cleaned_rows=6

expectation[no_empty_doc_id] OK (halt) :: empty_doc_id_count=0

expectation[refund_no_stale_14d_window] OK (halt) :: violations=0

expectation[chunk_min_length_8] OK (warn) :: short_chunks=0

expectation[effective_date_iso_yyyy_mm_dd] OK (halt) :: non_iso_rows=0

expectation[hr_leave_no_stale_10d_annual] OK (halt) :: violations=0

Warning: You are sending unauthenticated requests to the HF Hub. Please set a HF_TOKEN to enable higher rate limits and faster downloads.

Loading weights: 100%|██████████████████| 103/103 [00:00<00:00, 12737.74it/s]

BertModel LOAD REPORT from: sentence-transformers/all-MiniLM-L6-v2

Key                     | Status     |  | 

------------------------+------------+--+-

embeddings.position_ids | UNEXPECTED |  | 



Notes:

- UNEXPECTED:   can be ignored when loading from different task/architecture; not ok if you expect identical arch.

embed_upsert count=6 collection=day10_kb

manifest_written=artifacts/manifests/manifest_2026-04-15T08-38Z.json

freshness_check=FAIL {"latest_exported_at": "2026-04-10T08:00:00", "age_hours": 120.643, "sla_hours": 24.0, "reason": "freshness_sla_exceeded"}

PIPELINE_OK



(.venv) nxhai@nxhai:~/AI_thucchien/Lecture-Day-08-09-10/day10/lab$ python etl_pipeline.py run

run_id=2026-04-15T08-40Z

raw_records=10

cleaned_records=6

quarantine_records=4

cleaned_csv=artifacts/cleaned/cleaned_2026-04-15T08-40Z.csv

quarantine_csv=artifacts/quarantine/quarantine_2026-04-15T08-40Z.csv

expectation[min_one_row] OK (halt) :: cleaned_rows=6

expectation[no_empty_doc_id] OK (halt) :: empty_doc_id_count=0

expectation[refund_no_stale_14d_window] OK (halt) :: violations=0

expectation[chunk_min_length_8] OK (warn) :: short_chunks=0

expectation[effective_date_iso_yyyy_mm_dd] OK (halt) :: non_iso_rows=0

expectation[hr_leave_no_stale_10d_annual] OK (halt) :: violations=0

Warning: You are sending unauthenticated requests to the HF Hub. Please set a HF_TOKEN to enable higher rate limits and faster downloads.

Loading weights: 100%|██████████████████| 103/103 [00:00<00:00, 13634.63it/s]

BertModel LOAD REPORT from: sentence-transformers/all-MiniLM-L6-v2

Key                     | Status     |  | 

------------------------+------------+--+-

embeddings.position_ids | UNEXPECTED |  | 



Notes:

- UNEXPECTED:   can be ignored when loading from different task/architecture; not ok if you expect identical arch.

embed_upsert count=6 collection=day10_kb

manifest_written=artifacts/manifests/manifest_2026-04-15T08-40Z.json

freshness_check=FAIL {"latest_exported_at": "2026-04-10T08:00:00", "age_hours": 120.676, "sla_hours": 24.0, "reason": "freshness_sla_exceeded"}

PIPELINE_OK
**CSV**: Toàn bộ 4 câu hỏi kiểm thử (Refund, SLA, Lockout, Leave Policy) đều đạt contains_expected=yes và hits_forbidden=no.
| question_id | question | top1_doc_id | top1_preview | contains_expected | hits_forbidden | top1_doc_expected | top_k_used |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **q_refund_window** | Khách hàng có bao nhiêu ngày để yêu cầu hoàn tiền kể từ khi xác nhận đơn? | policy_refund_v4 | Yêu cầu được gửi trong vòng 7 ngày làm việc kể từ thời điểm xác nhận đơn hàng. | yes | no | | 3 |
| **q_p1_sla** | SLA phản hồi đầu tiên cho ticket P1 là bao lâu? | sla_p1_2026 | Ticket P1 có SLA phản hồi ban đầu 15 phút và resolution trong 4 giờ. | yes | no | | 3 |
| **q_lockout** | Bao nhiêu lần đăng nhập sai thì tài khoản bị khóa? | it_helpdesk_faq | Tài khoản bị khóa sau 5 lần đăng nhập sai liên tiếp. | yes | no | | 3 |
| **q_leave_version** | Theo chính sách nghỉ phép hiện hành (2026), nhân viên dưới 3 năm kinh nghiệm được bao nhiêu ngày phép năm? | hr_leave_policy | Nhân viên dưới 3 năm kinh nghiệm được 12 ngày phép năm theo chính sách 2026. | yes | no | yes | 3 |

Đặc biệt, câu hỏi về chính sách nghỉ phép năm 2026 trả về chính xác tài liệu hr_leave_policy với top1_doc_expected=yes.


---

## 5. Cải tiến thêm 2 giờ

Thay vì hard-code các ngưỡng thời gian (ví dụ: ngày hiệu lực chính sách 2026-01-01) trực tiếp trong code Python, tôi đã chuyển các tham số này vào file cấu hình ngoại vi. Việc này giúp hệ thống linh hoạt hơn, cho phép bộ phận HR hoặc Operations cập nhật chính sách mới mà không cần can thiệp vào mã nguồn của Pipeline AI.