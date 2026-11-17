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
| 2 | Nguyễn Thảo Quyên | 2A202603895 | thquyen100678@gmail.com | carwyn100678 | orchestrator + handoff |
| 3 |  Vũ Ngọc Quỳnh Anh  | 2A202603875 | 26ai.anhvnq@vinuni.edu.vn  | qavu2407-lang | single |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Vũ Ngọc Quỳnh Anh | single | `627a70f`, `3a0a638` | `src/tests/test_single_react.py` |
| Nguyễn Thảo Quyên | orchestrator + handoff | `e95d016`, `a9450b5`, `2c8190a` | `src/tests/test_orchestrator_extra.py` (57 test) |
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

## 2A202603895_NguyenThaoQuyen

### Phần việc phụ trách

- Shape C `orchestrator` (`src/lab8/shapes/orchestrator.py`): orchestrator, investigator chỉ có tool đọc, handoff allowlist. Làm trên nhánh `thaoquyen`.
- Test riêng của shape: `src/tests/test_orchestrator_extra.py`.
- `report/team/research_card.md` (câu R2, kèm ablation orchestrator so với swarm).
- Báo cáo cá nhân: `report/team/2A202603895_NguyenThaoQuyen.md`.

### Đóng góp cụ thể

| Checkpoint | Công việc của Thảo Quyên | Kết quả |
| --- | --- | --- |
| CP1 | Cài môi trường, chạy `smoke` và `pytest -q`; đọc luật nghiệp vụ, contract và `swarm_reference.py`. | Smoke pass; bộ student ra 186 failed, 62 passed như mốc đề bài. |
| CP2 | Viết `orchestrator.py`: handoff bằng `make_handoff` với ba tool đọc, `run_readonly_investigators`, quote bằng `compute_compensation` trên order row, ghi refund qua `durability.issue_refund` với quote đầy đủ. | Ma trận student của orchestrator: 62 passed. |
| CP3 | Truyền `crash_at` vào `issue_refund`, không bắt `InjectedCrash`. Trace chỉ ghi handoff đã sanitize, không ghi claim row hay ticket. | T11 và T12 xanh; trace sạch secret. |
| CP4 | Thêm 57 test: trace và handoff, crash ở 3 điểm rồi resume, approval hỏng hoặc mượn của claim khác, approval thấp hơn quote, trần theo đơn, ticket nói quá, không in ra stdout. | `test_orchestrator_extra.py`: 57 passed; không có `CANARY_SECRET_` trong trace. |
| CP5 | Điền `research_card.md` (R2) và chạy ablation orchestrator so với swarm. Viết báo cáo cá nhân. | Orchestrator pass rate 1,000, pass^5 1,000 trên 70 trial; swarm 0,986 và 0,929. |

- Chưa làm: tự kiểm trên `student_drop/` vì thư mục này do giảng viên phát cuối buổi.

### Bằng chứng

- Commit: `e95d016` (shape orchestrator), `a9450b5` (test riêng), `2c8190a` (research card và ablation), `b098933` và `af42422` (báo cáo cá nhân). PR #4 đã merge vào `main`.
- Test: `src/tests/test_orchestrator_extra.py`; `python -m pytest -q -m student -o addopts=` cho orchestrator ra 62 passed.
- Report sinh bằng công cụ: `src/reports/research_ablation_r2.json`.
