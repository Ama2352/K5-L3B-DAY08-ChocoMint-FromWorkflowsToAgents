# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3B
- Repo nhóm: `K5-L3A-DAY08-ChocoMint-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
| 1 | Hồ Nguyễn Minh Sang | 2A202603905 | 26ai.sanghnm@vinuni.edu.vn | Ama2352 | Nhóm trưởng; merge + build UI + nộp bài |
| 2 | | | | | |
| 3 |  Vũ Ngọc Quỳnh Anh  | 2A202603875 | 26ai.anhvnq@vinuni.edu.vn  | qavu2407-lang | single |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Vũ Ngọc Quỳnh Anh | single | `627a70f`, `3a0a638` | `src/tests/test_single_react.py` |
| Sang | Nhóm trưởng; merge + build UI + nộp bài | Chưa cập nhật | Chưa cập nhật test mới cho phần việc UI |

## Cá nhân

Mỗi thành viên tạo báo cáo riêng theo dạng `report/team/<MSSV>_<HoTen>.md`, ghi rõ công cụ AI đã dùng.

## 2A202603905_HoNguyenMinhSang

### Phần việc phụ trách

- Nhóm trưởng, làm việc trên nhánh `sang`; review và merge PR vào `main`, xây dựng UI và phụ trách hoàn thiện bài nộp.
- Theo phân công, Sang không viết shape. Workflow do Nhung phụ trách.
- File được sửa: `src/ui/` (mới), `TEAM.md`, `decision_adr.md`.
- Báo cáo cá nhân: `report/team/2A202603905_HoNguyenMinhSang.md`.

### Đóng góp cụ thể

Đây là kế hoạch được phân công trong [tasks/tasks.md](tasks/tasks.md), chưa phải xác nhận công việc đã hoàn thành.

| Checkpoint | Công việc của Sang | Mục tiêu |
| --- | --- | --- |
| CP1 (13:30) | Cùng nhóm cài môi trường, chạy `smoke` và `pytest -q`; đọc `business_rules.md`, `contracts.md`, `swarm_reference.py`. Fork repo, đổi tên theo phân công, mời 4 thành viên và giảng viên; điền `TEAM.md`. | Mốc kiểm tra ban đầu: 186 failed, 62 passed; hoàn thiện thông tin nhóm. |
| CP2 (13:45) | Dựng khung UI trong `src/ui/`, đọc `reports/*.json` dựa trên benchmark swarm có sẵn. | UI chạy được với dữ liệu mẫu. |
| CP3 (14:30) | Review và merge PR vào `main`; chủ trì cả nhóm chạy lại `pytest -q` trên `main`. | Tích hợp thay đổi và kiểm tra kết quả trên `main`. |
| CP4 (15:30) | Thêm màn hình crash safety vào UI; cùng nhóm xem kết quả crash test do Linh chạy. Quyết định và đăng ký bonus với giảng viên trước 15:30 nếu làm. | Hiển thị crash safety; mục tiêu crash test của nhóm: 20/20. |
| CP5 (16:15) | Cùng nhóm xem `benchmark.json`, thống nhất shape chọn và lý do. Điền `decision_adr.md` (commit hash, run id), cập nhật bảng đóng góp `TEAM.md`, chạy `check_submission` rồi push. | `check_submission` đạt 0 FAIL. |

- Viết báo cáo cá nhân, ghi công cụ AI đã dùng; nộp link repo trên LMS bằng tài khoản của mình.
- Làm trên nhánh riêng, mở PR vào `main`; trước mỗi PR chạy `git pull --rebase origin main` rồi `python -m pytest -q`.
- Không sửa tay `src/reports/*.json`. Có ít nhất một commit có nội dung và một test mới theo quy ước Git trong phân công.

### Bằng chứng

- Nhánh làm việc: `sang`; nhánh tích hợp và nộp bài: `main`.
- Commit/PR xây dựng UI, cập nhật tài liệu và tích hợp: chưa cập nhật.
- Test mới, kết quả chạy UI với dữ liệu mẫu và màn hình crash safety: chưa cập nhật.
- `decision_adr.md`, báo cáo cá nhân, kết quả `check_submission` và link nộp LMS: chưa cập nhật.
