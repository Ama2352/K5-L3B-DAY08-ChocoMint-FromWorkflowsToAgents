# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 6d9b8c248509ed1815269e3ff7e9a5c720f6426f
- Run id dẫn chứng: R-workflow-C-001-s11; R-workflow-C-002-s55; đối chứng R-swarm-C-002-s55 (src/reports/benchmark.json)
- Pass rate / pass^5 / seed min-max / variance: 1,000 / 1,000 / 1,000–1,000 / 0; public 14 claim × 5 seed = 70 trial workflow
- Cost per success (mô phỏng, VND): 34,2857142857 (hiển thị 34,29); tổng 2.400 VND / 70 trial đạt
- Crash safety: 20/20 trong src/reports/crash.json (suite chung host và một ca swarm, không phải 20 ca riêng cho workflow); retry/crash từng shape được kiểm thêm qua T11/T12 và test public
- Secret leak / approval bypass: test tích hợp 419 passed, 6 xfailed (các lỗi grader cố ý trong bonus B7); self-check 600 trial không lỗi thực thi/trace/cảnh báo canary; chưa có số đếm safety hidden chính thức hay hidden_grade.json

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Workflow, single và orchestrator cùng đạt 70/70 trial, pass⁵ = 1 và variance = 0. Workflow có cost/success thấp nhất: 34,29 so với single 39,41; orchestrator 47,72; swarm 49,57 VND mô phỏng. Tiết kiệm lần lượt khoảng 13,0%, 28,2%, 30,8% theo số gốc. Swarm còn fail C-002 seed 55 nên không tạo đánh đổi có lợi trong phép đo này. Vì logic hoàn tiền chủ yếu tuần tự và có luật rõ ràng, nhóm chọn workflow làm phương án ưu tiên trong phạm vi lab.

Thiết lập: public corpus 14 claim, seed 11,22,33,44,55, cùng as_of 2026-11-15, MockLLM, tool, pricing và evaluator; mỗi trial SQLite mới. UI dùng ngưỡng 100% public cho đề xuất kinh tế; đây không phải ngưỡng hidden 0,75 của đề bài.

## Có đáng đổi lấy multi-agent không

Chưa đáng đổi trong bộ public hiện tại: single/orchestrator tăng chi phí mà không tăng pass rate hay độ ổn định. Có thể đổi khi workflow không xử lý được tình huống mơ hồ/quy trình biến động, hoặc các phần việc độc lập giúp multi-agent cải thiện chất lượng/latency đủ bù chi phí. Khi đó phải đo lại trên tập đại diện bằng model thật trong cùng ranh giới an toàn; không suy ra lợi ích từ số agent đơn thuần.

## Trace và bằng chứng

Workflow ghi claim_view đã lọc (found/claim_id/order_id), lookup_order đã lọc, quote, refund, decision và crash khi có. Không ghi raw claim, ticket, parent_secret, raw_headers hay raw order có PII. Orchestrator ghi handoff sau allowlist, investigator chỉ có tool đọc. Mọi nhánh trả RunResult có trace_path tới JSON cùng run_id, events là list; trace nằm trong vùng scanner theo contract, không dùng chuyển vị trí để né quét. Các test public kiểm contract, chuỗi cấm, approval và ledger; không có hidden report để chứng minh vượt phạm vi đó.

Đối chiếu lỗi: R-swarm-C-002-s55 trả resolved thay vì denied do tin ticket ghi trễ 5 ngày trong khi order ghi 2 ngày. R-workflow-C-002-s55 trả denied. Bằng chứng chính là benchmark.json và research_ablation_r2.json; không commit trace chứa dữ liệu chạy.

## Giới hạn

MockLLM không phải model sản xuất. Latency `unavailable`; giá từ src/config/pricing.yaml, không phải hóa đơn API hoặc tổng chi phí doanh nghiệp. Public pass 100% không thay hidden ≥ 0,75. Nhóm đã nhận student_drop không nhãn; tự kiểm chỉ đánh giá thực thi/trace và nghi vấn canary, chưa có hidden_grade.json để kết luận điểm chính thức. So sánh orchestrator/swarm còn nhiễu số model call, không quy toàn bộ chênh lệch cho topology. Không sửa JSON để đủ ngưỡng.

Hash trên là snapshot mã nguồn shape dùng cho benchmark, không phải tuyên bố UI/tài liệu/JSON mới đã được commit hoặc push. Trước nộp cần chốt bản nguồn có thể chạy lại, kiểm tra hash/run ID và cập nhật nếu mã nguồn ảnh hưởng phép đo thay đổi.
