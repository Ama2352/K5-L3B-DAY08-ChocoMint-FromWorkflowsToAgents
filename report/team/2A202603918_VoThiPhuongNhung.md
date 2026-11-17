# Báo cáo cá nhân — Võ Thị Phương Nhung

## 1. Thông tin thành viên

- Họ tên: Võ Thị Phương Nhung
- MSSV: 2A202603918
- Email: nhungvo631@gmail.com
- GitHub: nhungvo631@gmail.com

## 2. Phần việc phụ trách

Em phụ trách Shape A — `workflow`, gồm triển khai workflow xác định, crash/retry, test edge case và bonus B7 kiểm toán grader.

## 3. Đóng góp cụ thể

- Hoàn thiện [`workflow.py`](../../src/lab8/shapes/workflow.py): đọc claim qua `get_claim_view`, tra cứu đơn bằng `lookup_order`, tự tính quote từ dữ liệu cấu trúc và gửi nguyên quote cho `issue_refund`. Ticket không quyết định số tiền; host kiểm approval, giới hạn theo đơn và ghi ledger.
- Truyền `crash_at` vào `issue_refund`; các test T11/T12 xác nhận retry không trả tiền hai lần và crash được ném tiếp.
- Thêm [`test_workflow_extra.py`](../../src/tests/test_workflow_extra.py) cho approval bị thu hồi, approval thuộc claim khác và đơn đã hoàn đủ.
- Làm bonus B7: viết [`grader_audit.py`](../../src/bonus/grader_audit.py), thêm [`test_bonus_grader_audit.py`](../../src/tests/test_bonus_grader_audit.py), sinh [`bonus_grader_audit.json`](../../src/reports/bonus_grader_audit.json) và viết [`bonus_report.md`](bonus_report.md). Phân tích ba grader bằng các fixture metamorphic có sẵn và quan hệ mới kiểm tra tính độc lập giữa shape.
- Kết quả B7: `grader_c` tính trial hạ tầng thành fail (pass rate 0.75 xuống 0.5625); cả ba grader làm `pass_5` từ 0.6667 xuống 0 khi nhân đôi fixture. `grader_a` còn làm lệch thống kê chi phí.
- Hoàn thiện bằng chứng nhóm bằng benchmark public 4 shape × 14 claim × 5 seed, crash suite 20 điểm và ADR. Từ benchmark, workflow đạt pass rate/pass^5 1.000/1.000, variance 0, cost/success 34.29 VND mô phỏng.
- CP5/hidden: không có bộ dữ liệu `student_drop/` hoặc kết quả chạy trong workspace hiện tại; vì vậy trạng thái và pass rate hidden chưa thể xác minh, không ghi số liệu.

## 4. Bằng chứng

Các lệnh dưới đây chạy từ thư mục `src/`:

- `python -m pytest -q -m student -o addopts= -k workflow`: 62 passed.
- `python -m pytest -q tests/test_workflow_extra.py`: 3 passed.
- `python -m pytest -q tests/test_bonus_grader_audit.py`: 10 passed.
- `python -m pytest -q`: 146 passed, 248 deselected.
- `python -m pytest -q -m student -o addopts=`: 248 passed, 146 deselected.
- `python -m lab8.check_submission`: 0 FAIL, 0 WARN.
- Benchmark: [`benchmark.json`](../../src/reports/benchmark.json), 70 trial cho mỗi shape; run id dẫn chứng workflow: `R-workflow-C-001-s11`.
- Crash suite: [`crash.json`](../../src/reports/crash.json), crash safety 20/20.

Các commit liên quan:

- `ed62c93` — workflow.
- `0c141e9` — test workflow bổ sung.
- `9da916d` — TEAM.md.
- `0a02a06` — bản báo cáo cá nhân trước.
- `4a275db` — cập nhật B7 và bằng chứng benchmark/crash.

## 5. Công cụ AI đã dùng

- Công cụ: Copilot SDK trong VS Code.
- Cách dùng: hỗ trợ triển khai workflow, tạo test edge case, xây dựng kiểm toán B7 và tổng hợp báo cáo.
- Tự kiểm tra: đối chiếu code với business rules và trace contract; chạy test workflow, test B7, toàn bộ suite và checker; kiểm tra kết quả audit được sinh trực tiếp từ fixtures.

## 6. Tự đánh giá

Phần workflow, crash/retry, test edge case và bonus B7 đã hoàn thành; toàn bộ checker và test trong workspace đều pass. Kết quả B7 chỉ áp dụng cho ba grader cùng các fixture hiện có, không chứng minh độ đúng trên mọi dữ liệu. Bonus gate về pass rate hidden ≥ 0.80 chưa xác minh được vì không có `student_drop/` trong workspace. Bản báo cáo này được viết lại sau commit `4a275db` và chưa có commit riêng.