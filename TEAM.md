# Team

## Thông tin nhóm

- Tên nhóm: ChocoMint (theo tên repository hiện tại).
- Lớp: L3A theo mẫu đề; tên repository hiện là L3B. Sang cần xác nhận lớp và quy ước tên repo với coach trước khi nộp.
- Repo nhóm: `K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents`; đối chiếu quy ước trong [SUBMISSION.md](SUBMISSION.md).
- URL repo, nhánh nộp, commit chốt: https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents ; nhánh nộp và commit chốt chờ Sang xác nhận. Phần Linh đang ở nhánh local `linh`, chưa commit/push/merge.

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
| 1 | Sang (chờ họ tên đầy đủ) | Chờ bổ sung | Chờ bổ sung | Chờ bổ sung | Merge code, UI, nộp bài |
| 2 | Nhung (chờ họ tên đầy đủ) | Chờ bổ sung | Chờ bổ sung | Chờ bổ sung | Shape A — Workflow |
| 3 | Quỳnh Anh (chờ họ tên đầy đủ) | Chờ bổ sung | Chờ bổ sung | Chờ bổ sung | Shape B — Single Agent |
| 4 | Thảo Quyên (chờ họ tên đầy đủ) | Chờ bổ sung | Chờ bổ sung | Chờ bổ sung | Shape C — Orchestrator + Handoff |
| 5 | Nguyễn Thị Khánh Linh | 2A202603845 | 26ai.linhntk@vinuni.edu.vn | [xrkhanhlinh](https://github.com/xrkhanhlinh/) | Swarm đối chứng (phân tích), Ledger, Crash, Benchmark, Report |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Thị Khánh Linh | Phân tích Swarm; kiểm chứng Ledger/Crash; benchmark và ablation; cập nhật báo cáo | Chưa commit, theo yêu cầu; `034c6c4` chỉ là HEAD nền, không phải commit đóng góp | `src/tests/test_linh_ledger_swarm.py`: 18 test case mới, đã pass |

Các thành viên khác bổ sung dòng đóng góp sau khi có bằng chứng thực tế; chưa khai commit hoặc test thay cho các bạn.

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603845_NguyenThiKhanhLinh

### Phần việc phụ trách

- Shape D chỉ phân tích; kiểm Ledger, approval và crash; benchmark Swarm, research card, phần số liệu báo cáo.

### Đóng góp cụ thể

- Thêm 18 test case hồi quy trong `src/tests/test_linh_ledger_swarm.py` và script `src/scripts/evaluate_swarm_linh.py`; không sửa shape/host/config/data.
- Chạy lại smoke, host gate, ma trận student riêng Swarm, demo lỗi và crash suite. Tổng host sau bổ sung: 72 passed; Swarm student: 62 passed.
- Script sinh lại file `src/reports/benchmark_swarm.json` có sẵn, giữ các khóa benchmark chuẩn và thêm `linh_public_safety`, `linh_ablation`. Không tạo file báo cáo mới.

### Bằng chứng

- `src/reports/benchmark_swarm.json`: baseline 69/70 trial, pass^5 13/14; ablation bỏ riêng lời gọi peer vẫn 69/70, giảm cost/success.
- `src/reports/crash.json`: 20/20, phạm vi host và Swarm. Báo cáo chi tiết: [báo cáo cá nhân](report/team/2A202603845_NguyenThiKhanhLinh.md).
- Còn chờ: A/B/C merge, benchmark bốn shape, quyết định cuối và commit đóng góp. Chưa thực hiện commit/push/merge.
