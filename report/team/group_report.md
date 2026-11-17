# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: ChocoMint.
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): chưa có; chờ tích hợp và được phép commit. HEAD nền `034c6c4` không phải commit chứa phần việc mới. Phần Linh ở nhánh local `linh`.

## 2. Mục tiêu và cách tiếp cận

Mục tiêu là xử lý claim hoàn phí/hỏng hàng bằng cùng contract và chọn kiến trúc ít tự chủ nhất vẫn đạt yêu cầu. Tiền, approval và ghi ledger do code quyết; ticket là dữ liệu không đáng tin. So sánh phải giữ cùng corpus, seed, MockLLM, tool và bảng giá.

Sang phụ trách tích hợp/UI/nộp; Nhung viết Workflow; Quỳnh Anh viết Single Agent; Thảo Quyên viết Orchestrator/Handoff; Linh phân tích Swarm đối chứng, Ledger, Crash, Benchmark và Report. Ở checkout đang kiểm tra, A/B/C chưa merge và vẫn là stub; báo cáo này chỉ chốt số liệu Swarm/host đã thực thi, chưa tuyên bố hoàn thành phần của các thành viên khác.

## 3. Giải pháp

Workflow dự kiến đi các bước cố định; Single dùng ReAct tối đa 8 bước; Orchestrator giao investigator chỉ đọc qua handoff allowlist. Mô tả implementation thực tế của A/B/C chờ các tác giả bổ sung sau merge.

Swarm có sẵn gọi MockLLM cho peer/parse/draft, tạo handoff đã sanitize, investigator đọc order/policy và gọi `compute_compensation`; shape chuyển quote đầy đủ cho `issue_refund`. Host kiểm ownership, approval đúng claim/còn hạn, trần approval/paid và tổng tiền theo đơn. Ledger có khóa `refund:{claim_id}`; refund và audit committed chung transaction, checkpoint ghi sau. T14 quote 150.000, đã hoàn 30.000 nên chỉ ghi effect 120.000; không trừ lịch sử hai lần.

Trace chỉ chứa handoff và dữ kiện được phép; bỏ secret, header, phone/address/internal note/item name. Lỗi cố ý `REFERENCE_WEAKNESS` của Swarm thay delay bằng số trong ticket khi seed chia hết cho 5, làm sai T02 seed 55.

Linh thêm 18 test hồi quy trong `src/tests/test_linh_ledger_swarm.py` và script tái lập `src/scripts/evaluate_swarm_linh.py`. Script audit từng public trial và ablate riêng lời gọi peer bằng patch tạm trong process; không sửa shape/host/config/data. Kết quả ghi vào JSON benchmark Swarm đã có, với các khóa bổ sung `linh_public_safety` và `linh_ablation`.

## 4. Kết quả

Kết quả chạy ngày 17/11/2026 (GMT+7): smoke pass; host trước/sau thêm test lần lượt 54/72 passed; test cá nhân 18 passed; student riêng Swarm 62 passed; demo cố ý lỗi tạo 2 dòng hoàn tiền. Đây là output lệnh thực tế; các số benchmark bên dưới lấy từ `src/reports/benchmark_swarm.json` do script sinh.

| Shape / nhánh thử | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean/p95 steps |
| --- | ---: | ---: | --- | ---: | ---: | --- |
| Workflow | Chờ merge | Chưa đo | Chưa đo | Chưa đo | Chưa đo | Chưa đo |
| Single | Chờ merge | Chưa đo | Chưa đo | Chưa đo | Chưa đo | Chưa đo |
| Orchestrator | Chờ merge | Chưa đo | Chưa đo | Chưa đo | Chưa đo | Chưa đo |
| Swarm gốc | 69/70 = 0,985714 | 13/14 = 0,928571 | 0,928571–1 | 0,000816327 | 49,566957 | 2,8 / 3 |
| Swarm bỏ peer (ablation) | 69/70 = 0,985714 | 13/14 = 0,928571 | 0,928571–1 | 0,000816327 | 48,691594 | 2,8 / 3 |

Mỗi nhánh dùng 14 claim × 5 seed `11,22,33,44,55`, DB mới mỗi trial. Tổng cost gốc/bỏ peer là 3420,12/3359,72 VND mô phỏng; mean model calls 3/2, tool calls giữ 4,657143. Cặp business outcomes và pass/fail giống nhau. Steps không giảm vì evaluator đếm event trace. Lỗi duy nhất `R-swarm-C-002-s55` trả tiền vì tin ticket; test student coi lỗi đối chứng này là expected miss, nên 62 test pass vẫn tương thích benchmark 69/70.

`src/reports/crash.json` ghi 20/20; suite kiểm host và một scenario Swarm, chưa chứng minh crash của A/B/C. Audit local trên 70 trial mỗi nhánh ghi 0 secret leak, 0 giá trị nhạy cảm public bị lộ, 0 approval bypass, 0 ledger ngoài claim, 0 trace sai và 0 trial chưa audit; không phải hidden grade. Chưa có `student_drop`/hidden grade để kết luận ngưỡng hidden.

Lệnh tái lập từ `src/` (activate `.venv` ở root, hoặc dùng `..\.venv\Scripts\python.exe`):

```text
python -m lab8.smoke
python -m pytest -q
python -m pytest -q tests/test_linh_ledger_swarm.py
python -m pytest -q -m student -o addopts= -k swarm
python -m lab8.demo.buggy_workflow
python -m lab8.crash_test --count 20 --out reports/crash.json
python -m scripts.evaluate_swarm_linh
```

CLI chuẩn `python -m lab8.benchmark --shapes swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark_swarm.json` chỉ ghi baseline. Chạy script của Linh sau lệnh này để sinh cả audit và ablation vào cùng file; không sửa tay JSON.

Đã chạy lại `scripts.evaluate_swarm_linh` lần hai: SHA256 của toàn bộ JSON trước/sau trùng nhau (baseline, audit, ablation). Đây là kiểm chứng tái lập trên cùng máy/working tree, chưa phải kiểm chứng từ commit sạch hoặc trên máy khác.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Checkout ban đầu ở `main`, chưa có ref `linh` trong danh sách local/remote đã lưu; đã tạo local `linh` từ HEAD nền, mang theo working tree hiện có. Không fetch, commit, push hoặc merge. Python/pytest/dependency đang dùng đều chạy được, chưa ghi nhận lỗi cài đặt.

Ba shape stub là dependency tích hợp, không sửa thay tác giả và không xem pass rate của stub là hiệu năng kiến trúc. Không chọn Swarm chỉ để làm `check_submission` xanh. `Shape chọn`/`Commit hash` trong ADR còn trống có chủ đích. File mẫu `report/team/MSSV_HoTen.md` đã bị xóa trước lượt làm việc; giữ nguyên thay đổi sẵn có này, Sang cần khôi phục mẫu tracked trước khi nộp theo README.

Đã chạy `python -m lab8.check_submission`: **6 FAIL, 1 WARN**. Sáu mục là ba shape `not_implemented`, thiếu `reports/benchmark.json`, chưa chọn shape và chưa có commit hash. Cảnh báo là working tree chưa commit. Research card, bảng đóng góp và crash report đã được checker đọc thành công; kết quả này chưa đủ điều kiện nộp.

Việc cần làm sau merge, theo thứ tự:

1. Chạy `python -m pytest -q` và `python -m pytest -q -m student -o addopts=`; kiểm T11/T12 của từng shape, approval, trace/handoff.
2. Chạy `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`.
3. Điền bảng so sánh thực tế, chọn shape ở ADR, ghi run id; bổ sung tự kiểm hidden nếu có pack không nhãn. Hidden pass rate chính thức do giảng viên chấm.
4. Sang xác nhận lớp/tên repo (mẫu đề L3A, repo hiện L3B), metadata thành viên và nhánh nộp. Linh tự kiểm tra nội dung và tự đánh giá cá nhân.
5. Khi được phép, tạo commit thật và ghi hash đóng góp/chốt, khôi phục mẫu báo cáo; chạy `python -m lab8.check_submission` đến 0 FAIL trước bước nộp. Chưa thực hiện commit/push/merge.

## 6. Tổng hợp đóng góp

Chi tiết phân công và đóng góp đã kiểm chứng nằm trong `TEAM.md` và báo cáo `report/team/2A202603845_NguyenThiKhanhLinh.md`. Phần Linh gồm 18 test, harness tái lập, hai file bằng chứng JSON có sẵn và nội dung báo cáo; chưa có commit đóng góp. Các thành viên khác tự bổ sung bằng chứng sau merge.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`, giá là mô phỏng. Public chỉ có 14 claim và Swarm cố ý có lỗi. Bỏ lời gọi peer có output bị bỏ qua cho thấy overhead trong code hiện tại, không chứng minh các topology multi-agent khác đều thừa. Audit chỉ kiểm các giá trị public/kênh đã nêu. Chưa có số liệu A/B/C hoặc hidden, nên chưa quyết định kiến trúc cuối cùng. Cần đo lại khi peer output tham gia quyết định, dữ liệu/task đổi hoặc dùng model thật trong một nghiên cứu khác ngoài lab.

## 8. Tài liệu tham khảo

- [Luật nghiệp vụ](../../docs/business_rules.md), [contract](../../docs/contracts.md), [metrics](../../docs/metrics.md), [rubric](../../RUBRIC.md), [rules](../../RULES.md).
- Mã nguồn `src/lab8/shapes/swarm_reference.py`, `durability.py`, `policy.py`, `metrics.py`; JSON do công cụ sinh trong `src/reports/`.
- Nguồn gốc của research card: Zhang và cộng sự (EMNLP 2026), [Rethinking the Evaluation of Efficiency Methods for Multi-Agent Systems](https://aclanthology.org/2026.emnlp-main.798/). Claim và giới hạn sử dụng nguồn được ghi tại `research_card.md`.
