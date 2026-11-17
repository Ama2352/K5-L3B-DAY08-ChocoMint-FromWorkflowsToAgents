# Team

## Thông tin nhóm

- Tên nhóm: ChocoMint
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo: https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/
- Nhánh nộp: `main`.
- Commit mã nguồn đối chiếu: `f526841f182ecec31305ddf6db0750026f6d440a`, trùng ADR; báo cáo bổ sung được commit sau snapshot này. Lớp L3B đã được nhóm trưởng xác nhận; mẫu SUBMISSION ghi L3A.

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
| 1 | Võ Thị Phương Nhung | 2A202603918 | nhungvo631@gmail.com | nhungvo631@gmail.com | workflow, crash, hidden self-check |
| 2 | Hồ Nguyễn Minh Sang | 2A202603905 | 26ai.sanghnm@vinuni.edu.vn | Ama2352 | Nhóm trưởng; merge + build UI + nộp bài |
| 3 | Nguyễn Thảo Quyên | 2A202603895 | thquyen100678@gmail.com | carwyn100678 | orchestrator + handoff |
| 4 |  Vũ Ngọc Quỳnh Anh  | 2A202603875 | 26ai.anhvnq@vinuni.edu.vn  | qavu2407-lang | single |
| 5 | Nguyễn Thị Khánh Linh | 2A202603845 | 26ai.linhntk@vinuni.edu.vn | xrkhanhlinh | Swarm (đối chứng, chỉ phân tích), Ledger, Crash, Benchmark và Report |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Võ Thị Phương Nhung | workflow, crash, hidden self-check | `ed62c93`, `0c141e9`, `9da916d`, `0a02a06` | `src/tests/test_workflow_extra.py`: approval bị thu hồi, approval thuộc claim khác, đơn đã hoàn đủ; ma trận workflow T01–T14/T11–T12 |
| Vũ Ngọc Quỳnh Anh | single | `627a70f`, `3a0a638`, `176a26b` | `src/tests/test_single_react.py`, `src/tests/test_single_extra.py` |
| Nguyễn Thảo Quyên | orchestrator + handoff | `e95d016`, `a9450b5`, `2c8190a` | `src/tests/test_orchestrator_extra.py` (57 test) |
| Hồ Nguyễn Minh Sang | Nhóm trưởng; tích hợp/merge, xây UI benchmark và crash safety, chuẩn bị báo cáo và bài nộp | `5022b6f`, `9705de5`, `8dc0a54` | `src/ui/report.test.mjs`, `src/ui/integration.test.mjs`: kiểm cấu trúc báo cáo, phân biệt trial fail/chưa chấm, chọn shape theo chất lượng và chi phí trên cùng claim × seed, đối chiếu chỉ số với JSON thật; lệnh: `node --test src/ui/report.test.mjs src/ui/integration.test.mjs` |
| Nguyễn Thị Khánh Linh | Swarm đối chứng, ledger, crash, benchmark và báo cáo | `268d137` (nguồn gốc); `f526841` (tích hợp nguyên test/harness/report vào main, giữ tác giả); `be7007e` (báo cáo, PR #12) | `src/tests/test_linh_ledger_swarm.py`: 18 test case; có trên nhánh nộp main |

## Cá nhân

Mỗi thành viên tạo báo cáo riêng theo dạng `report/team/<MSSV>_<HoTen>.md`, ghi rõ công cụ AI đã dùng.

## 2A202603905_HoNguyenMinhSang

### Phần việc phụ trách

- Nhóm trưởng, làm việc trên nhánh `sang`; review và merge PR vào `main`, xây dựng UI và phụ trách hoàn thiện bài nộp.
- Theo phân công, Sang không viết shape. Workflow do Nhung phụ trách.
- File đóng góp: `src/ui/`, `TEAM.md`, `report/team/decision_adr.md`, `report/team/group_report.md`, `docs/demo_script.md` và báo cáo cá nhân.
- Báo cáo cá nhân: `report/team/2A202603905_HoNguyenMinhSang.md`.

### Đóng góp cụ thể

Kế hoạch CP dưới đây được đối chiếu với bằng chứng thực tế trong [báo cáo cá nhân Sang](report/team/2A202603905_HoNguyenMinhSang.md): `5022b6f` (UI/test/báo cáo), `9705de5` (demo) và các merge commit. Chưa suy ra việc đã nộp LMS từ kế hoạch.

| Checkpoint | Công việc của Sang | Mục tiêu |
| --- | --- | --- |
| CP1 (13:30) | Cùng nhóm cài môi trường, chạy `smoke` và `pytest -q`; đọc `business_rules.md`, `contracts.md`, `swarm_reference.py`. Fork repo, đổi tên theo phân công, mời 4 thành viên và giảng viên; điền `TEAM.md`. | Mốc kiểm tra ban đầu: 186 failed, 62 passed; hoàn thiện thông tin nhóm. |
| CP2 (13:45) | Dựng khung UI trong `src/ui/`, đọc `reports/*.json` dựa trên benchmark swarm có sẵn. | UI chạy được với dữ liệu mẫu. |
| CP3 (14:30) | Review và merge PR vào `main`; chủ trì cả nhóm chạy lại `pytest -q` trên `main`. | Tích hợp thay đổi và kiểm tra kết quả trên `main`. |
| CP4 (15:30) | Thêm màn hình crash safety vào UI; đối chiếu báo cáo crash suite của nhóm. Quyết định và đăng ký bonus với giảng viên trước 15:30 nếu làm. | Hiển thị crash safety; mục tiêu crash test của nhóm: 20/20. |
| CP5 (16:15) | Cùng nhóm xem `benchmark.json`, thống nhất shape chọn và lý do. Điền `decision_adr.md` (commit hash, run id), cập nhật bảng đóng góp `TEAM.md`, chạy `check_submission` rồi push. | `check_submission` đạt 0 FAIL. |

## 2A202603918_VoThiPhuongNhung

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
- CP5: báo cáo nhóm `src/reports/hidden_selfcheck.json` đã có trên main qua PR #14: workflow 200 trial, toàn bộ reason là `unlabeled`, không lỗi thực thi/trace hoặc cảnh báo canary. Không phải điểm hidden chính thức.
- Commit: `ed62c93` (workflow), `0c141e9` (extra tests), `9da916d` (TEAM.md), `0a02a06` (báo cáo cá nhân)
- Viết báo cáo cá nhân, ghi công cụ AI đã dùng; nộp link repo trên LMS bằng tài khoản của mình.
- Làm trên nhánh riêng, mở PR vào `main`; trước mỗi PR chạy `git pull --rebase origin main` rồi `python -m pytest -q`.
- Không sửa tay `src/reports/*.json`. Có ít nhất một commit có nội dung và một test mới theo quy ước Git trong phân công.

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

- CP5 tự kiểm trên `student_drop/`: orchestrator chạy 200 trial (40 claim × 5 seed), 0 `error`, 0 `leak_suspect`; không có pass rate vì bộ không có nhãn. Kết quả ở `src/reports/hidden_selfcheck.json`.

### Bằng chứng

- Commit: `e95d016` (shape orchestrator), `a9450b5` (test riêng), `2c8190a` (research card và ablation), `b098933` và `af42422` (báo cáo cá nhân). PR #4 đã merge vào `main`.
- Test: `src/tests/test_orchestrator_extra.py`; `python -m pytest -q -m student -o addopts=` cho orchestrator ra 62 passed.
- Report sinh bằng công cụ: `src/reports/research_ablation_r2.json`.

## 2A202603845_NguyenThiKhanhLinh

### Phần việc phụ trách

- Shape D — Swarm tham chiếu: chỉ phân tích và làm đối chứng; không sửa implementation của các shape.
- Ledger, idempotency, approval, crash/retry, benchmark riêng Swarm và phần báo cáo tương ứng.
- Báo cáo cá nhân: `report/team/2A202603845_NguyenThiKhanhLinh.md`.

### Đóng góp cụ thể

- Phân tích ranh giới code/model: tiền do `compute_compensation` tính; host kiểm approval, trần hoàn tiền theo đơn và chống ghi trùng. Giải thích lỗi đối chứng T02 ở seed 55 khi Swarm tin số ngày trong ticket.
- Thêm 18 test case trong `src/tests/test_linh_ledger_swarm.py`: crash ở ba điểm với refund bị cap/hoàn phần chênh; reopen và retry; biên hết hạn và approval không hợp lệ; conflict khi quote đổi; thu hồi approval sau thanh toán; trace/console không lộ dữ liệu nhạy cảm; ablation và phục hồi runtime.
- Thêm `src/scripts/evaluate_swarm_linh.py` để sinh baseline, audit public và ablation bỏ riêng lời gọi peer. Giữ nguyên code host, Swarm tham chiếu, fixture, giá và evaluator.
- Chạy smoke, test cá nhân, ma trận Swarm, demo workflow cố ý lỗi và crash suite. Benchmark cùng 14 claim public và seed `11,22,33,44,55`, không tạo số liệu giả hoặc sửa tay JSON.
- Viết báo cáo cá nhân và phần phân tích Swarm/Ledger/Crash/Benchmark. Báo cáo cá nhân đã được đưa vào `main` qua PR #12; phần tích hợp, kiểm thử cuối và chốt bài nộp do Sang phối hợp với nhóm xử lý.

### Bằng chứng

- Commit kết quả: [268d137](https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/commit/268d137), đã push lên nhánh `linh`.
- [Test cá nhân](https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/blob/linh/src/tests/test_linh_ledger_swarm.py): 18 passed; test cá nhân cũng đã được chạy trên bản tích hợp `394d35b` trong checkout tạm và đạt 18 passed. Ma trận student riêng Swarm: 62 passed.
- Crash suite: 20/20. Demo cố ý lỗi tạo 2 dòng hoàn tiền, minh họa checkpoint đơn độc không thay thế ledger.
- [Benchmark Swarm và ablation](https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/blob/linh/src/reports/benchmark_swarm.json): hai nhánh đều 69/70 trial pass, pass^5 13/14; bỏ peer giảm cost/success từ 49,566957 xuống 48,691594 VND mô phỏng. Audit local public trên 70 trial mỗi nhánh ghi 0 secret leak, 0 lộ giá trị nhạy cảm public và 0 approval bypass; không phải hidden grade.
- Báo cáo cá nhân: commit `be7007e`, [PR #12](https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/pull/12), merge commit `9b867dc` trên `main`.
- Test, harness và benchmark được tích hợp nguyên bản từ `268d137` vào `main` tại `f526841`, giữ tác giả gốc. PR #12 trước đó chỉ đưa báo cáo cá nhân vào main.
