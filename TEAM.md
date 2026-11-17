# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-TenNhom-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Võ Thị Phương Nhung | 2A202603918 | nhungvo631@gmail.com | nhungvo631@gmail.com | workflow, crash, hidden self-check |
|   2 |        |      |       |        |         |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Võ Thị Phương Nhung | workflow, crash, hidden self-check | Chưa commit | `src/tests/test_workflow_extra.py`: approval bị thu hồi, approval thuộc claim khác, đơn đã hoàn đủ; ma trận workflow T01–T14/T11–T12 |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## Võ Thị Phương Nhung

### Phần việc phụ trách

- Hoàn thiện shape workflow, xử lý crash/retry, bổ sung test edge case và tự kiểm shape trên `student_drop/` (CP5).

### Đóng góp cụ thể

- Triển khai luồng workflow xác định, tính quote từ dữ liệu hệ thống và chuyển quote đầy đủ cho host xử lý.
- Chuyển `crash_at` tới `issue_refund`; ma trận workflow xác nhận T11/T12 xanh.
- Bổ sung test cho approval bị thu hồi, approval thuộc claim khác và đơn đã hoàn đủ.
- Tự kiểm shape workflow trên bộ `student_drop/` theo CP5.

### Bằng chứng

- `src/lab8/shapes/workflow.py`
- `src/tests/test_workflow_extra.py` — 3 test mới passed; ma trận workflow — 62 test passed, bao gồm T11/T12.
- CP5: đã tự kiểm trên `student_drop/`; thư mục dữ liệu và kết quả không có trong workspace hiện tại nên không ghi số liệu.
- Commit: chưa commit.
