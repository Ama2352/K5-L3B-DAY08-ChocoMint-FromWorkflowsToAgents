# Báo cáo cá nhân — 2A202603905_HoNguyenMinhSang

## 1. Thông tin thành viên

- Họ tên: Hồ Nguyễn Minh Sang
- MSSV: 2A202603905
- Email: 26ai.sanghnm@vinuni.edu.vn
- GitHub: Ama2352
- Vai trò: nhóm trưởng, tích hợp, xây UI và chuẩn bị bài nộp.

## 2. Phần việc phụ trách

Tích hợp các shape của thành viên, xây UI đọc báo cáo benchmark/crash, viết test UI, tổng hợp ADR, báo cáo nhóm và kịch bản demo. Không nhận phần triển khai ba shape là đóng góp cá nhân: workflow do Nhung, single do Quỳnh Anh, orchestrator và nghiên cứu R2 do Thảo Quyên phụ trách.

## 3. Đóng góp cụ thể

| Phần việc | Kết quả và bằng chứng |
| --- | --- |
| Tích hợp | Các merge commit `5b49c94` (PR #9), `24037cc` (PR #10), `89f0b72` (main vào Sang), `8dc0a54` (PR #11). |
| UI benchmark và crash | `5022b6f`: `src/ui/`, phân biệt báo cáo thật/mẫu, biểu đồ chất lượng và chi phí, so cùng claim/seed, bộ lọc và crash safety. |
| Logic lựa chọn | Chỉ đề xuất shape đạt ngưỡng chất lượng trên cùng claim × seed; tính mức tiết kiệm theo baseline. Chi phí thấp không chứng minh đã qua safety/hidden. |
| Test | `src/ui/report.test.mjs`, `src/ui/integration.test.mjs` trong `5022b6f`: kiểm format, trial chưa chấm, bộ lọc, điều kiện so sánh và chỉ số từ JSON thật. |
| Báo cáo và demo | `5022b6f` bổ sung báo cáo benchmark/crash, ADR và báo cáo nhóm; `9705de5` bổ sung script demo, kiến thức CP1–CP5 và vấn đáp. |

Quyết định kỹ thuật: dùng UI tĩnh đọc JSON do công cụ lab sinh; thao tác đổi claim/seed không chạy shape mới. Chi phí là VND mô phỏng, steps là event trace; không suy ra latency. Không đưa raw claim/order, secret hoặc PII lên UI và không sửa tay số liệu JSON.

## 4. Bằng chứng

- [Commit UI và test 5022b6f](https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/commit/5022b6f).
- [Commit script demo 9705de5](https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents/commit/9705de5).
- `src/reports/benchmark.json`: workflow/single/orchestrator mỗi shape 70/70, swarm 69/70. Đây là kết quả nhóm, không phải số test cá nhân.
- Ca demo: `R-workflow-C-002-s55` từ chối đúng; `R-swarm-C-002-s55` chấp nhận sai. Workflow cost/success khoảng 34,29 VND mô phỏng, thấp hơn single khoảng 13% trên public.
- `src/reports/crash.json`: suite chung 20/20, không phải 20 ca riêng cho từng shape.

Test UI chạy từ gốc repo:

```powershell
node --test src/ui/report.test.mjs src/ui/integration.test.mjs
```

Lần kiểm tra cuối khi rà hồ sơ ngày 17/11/2026 đạt **9 test UI**, khớp bộ test hiện có trong workspace và commit `5022b6f`. Bản mở rộng giải thích T02 từng được kiểm trong phiên trước nhưng không còn trong workspace ở lần rà này, nên không tính vào bằng chứng nộp hiện tại.

Kiểm tra tích hợp chạy từ `src/`, sau khi kích hoạt môi trường:

```powershell
python -m pytest -q -o addopts= -p no:cacheprovider
python -m lab8.check_submission
```

Sau tích hợp PR #12–#16 và test/harness của Linh (`f526841`), toàn bộ pytest đạt **419 passed, 6 xfailed**. Các xfail là lỗi grader cố ý trong bonus B7, không phải lỗi shape. Đã chạy lại benchmark chuẩn, crash suite 20/20 và báo cáo bonus bằng script. Self-check 600 trial đã có trên main qua PR #14; không có lỗi thực thi/trace/cảnh báo canary nhưng không chứng minh hidden pass rate hoặc approval bypass chính thức.

Sang xử lý conflict TEAM khi đồng bộ main, giữ đầy đủ năm thành viên, tích hợp bằng chứng kỹ thuật của Linh và cập nhật hồ sơ theo PR mới. Merge tích hợp #12–#14 là `2b4859a`; báo cáo bonus #15 cũng được tích hợp. PR #16 được xử lý conflict: giữ hai bộ test B7, tách JSON isolation/permutation và nhận bản sửa checker tên topic B1–B7. Báo cáo chính ở `bonus_report.md`; gate hidden bonus vẫn chờ. Sang xác nhận lớp chính thức L3B và đã nộp link repo trên LMS; chưa có xác nhận nộp LMS của bốn thành viên khác.

## 5. Công cụ AI đã dùng

Đã dùng ChatGPT/Codex trong phiên này để đọc yêu cầu, phân tích pytest, bổ sung UI/test, tổng hợp demo, rà hồ sơ và soạn báo cáo từ file cùng lịch sử Git. Codex đã đọc code/JSON, chạy test và kiểm diff; các thao tác đó không đồng nghĩa cá nhân đã tự kiểm tra thủ công.

Trước khi nộp, Sang cần đọc lại code và báo cáo, chạy ca demo trên trình duyệt, đối chiếu số liệu với JSON và giải thích được nguồn dữ liệu, logic shortlist, quote/refund cùng giới hạn MockLLM. Chỉ bổ sung công cụ hoặc thao tác tự kiểm khác nếu thực sự đã dùng/làm.

## 6. Tự đánh giá

Phạm vi có bằng chứng là tích hợp, UI, test và tài liệu. UI phục vụ trình bày kết quả đã chạy; chưa phải backend thực thi claim trực tiếp. Public và MockLLM chưa chứng minh hiệu quả production. Phần nhận xét cá nhân về điều đã học và mức độ hoàn thành cần Sang xác nhận trước khi nộp; bản tổng hợp này không tự gán điểm số hay trải nghiệm cá nhân.
