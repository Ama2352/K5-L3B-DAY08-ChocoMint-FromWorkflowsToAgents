# Báo cáo cá nhân — Võ Thị Phương Nhung

## 1. Thông tin thành viên

- Họ tên: Võ Thị Phương Nhung
- MSSV: 2A202603918
- Email: nhungvo631@gmail.com
- GitHub: nhungvo631@gmail.com

## 2. Phần việc phụ trách

Shape A — workflow trong [`src/lab8/shapes/workflow.py`](../../src/lab8/shapes/workflow.py), xử lý crash/retry, kiểm thử edge case và tự kiểm shape trên bộ `student_drop/` (CP5).

## 3. Đóng góp cụ thể

- Hoàn thiện workflow theo luồng xác định: đọc claim, tra cứu đơn, tự tính quote từ dữ liệu hệ thống và gửi nguyên quote cho `issue_refund`.
- Giữ quyết định về khoản tiền trong code, không tin số ngày ghi trong ticket; xử lý các nhánh thiếu thông tin, không thấy đơn, quote bằng 0, bị host từ chối và thành công.
- Chuyển `crash_at` tới `issue_refund`, không nuốt lỗi crash; kiểm tra retry/crash qua T11/T12.
- Thêm [`src/tests/test_workflow_extra.py`](../../src/tests/test_workflow_extra.py), kiểm tra approval bị thu hồi, approval thuộc claim khác và đơn đã hoàn đủ.
- Tự kiểm shape workflow trên bộ `student_drop/` theo CP5.
- Commit chính: Chưa commit.

## 4. Bằng chứng

- `python -m pytest -q -m student -o addopts= -k workflow` (chạy từ `src/`): 62 passed.
- `python -m pytest -q tests/test_workflow_extra.py` (chạy từ `src/`): 3 passed.
- `python -m lab8.check_submission` xác nhận workflow đạt 12/12 claim public ở seed 11. CP5: không có thư mục `student_drop/` hay kết quả tự kiểm trong workspace hiện tại để ghi số liệu.
- Các thay đổi tương ứng nằm trong [`workflow.py`](../../src/lab8/shapes/workflow.py) và [`test_workflow_extra.py`](../../src/tests/test_workflow_extra.py).

## 5. Công cụ AI đã dùng

- Công cụ: Copilot SDK trong VS Code.
- Cách dùng: hỗ trợ triển khai workflow, bổ sung test và đối chiếu với hợp đồng của lab.
- Tự kiểm tra: chạy ma trận workflow và test bổ sung; kiểm tra audit từ chối và không phát sinh refund cho ba trường hợp bị từ chối. Phần CP5 được ghi nhận theo thông tin thực hiện; kết quả hidden không có trong workspace để đối chiếu lại.

## 6. Tự đánh giá

Phần workflow, crash/retry, ba trường hợp kiểm tra bổ sung và tự kiểm CP5 đã hoàn thành theo phân công; các test workflow có sẵn trong workspace đều pass. Chưa có mã commit để ghi nhận. Bộ test mặc định của repository còn lỗi ở các shape `single` và `orchestrator`, ngoài phạm vi phần việc này.
