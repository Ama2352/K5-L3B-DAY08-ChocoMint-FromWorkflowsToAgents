# Báo cáo cá nhân — 2A202603845_NguyenThiKhanhLinh

Trạng thái cập nhật ngày 17/11/2026 (GMT+7): báo cáo đã merge qua PR #12; test/harness và benchmark riêng được tích hợp nguyên bản từ `268d137` vào main tại `f526841`, giữ tác giả gốc. Các số liệu ở phần quá trình bên dưới là snapshot lịch sử, không phải trạng thái cuối nhánh nộp.

## 1. Thông tin thành viên

- Họ tên: Nguyễn Thị Khánh Linh
- MSSV: 2A202603845
- Email: 26ai.linhntk@vinuni.edu.vn
- GitHub: https://github.com/xrkhanhlinh/

## 2. Phần việc phụ trách

Shape D – Swarm (đối chứng, chỉ phân tích).
Phụ trách Ledger + Crash + Benchmark + Báo cáo.


## 3. Đóng góp cụ thể

Commit đóng góp gốc: `268d137` trên nhánh `linh`; bản tích hợp kỹ thuật trên main là `f526841`. Báo cáo được đưa vào main qua PR #12 (`be7007e`, merge `9b867dc`). Commit nền `034c6c4` là mã đề, không phải đóng góp cá nhân.

- Bổ sung `src/tests/test_linh_ledger_swarm.py`, gồm 18 test case: 6 tổ hợp crash/reopen/retry cho refund bị cap và hoàn phần chênh; 2 biên ngày hết hạn approval; 4 loại approval không hợp lệ; conflict sau crash khi quote đổi; thu hồi approval sau khi đã trả; 3 nhánh trace/console; 1 kiểm chứng ablation và phục hồi runtime.
- Giữ quote đầy đủ: T08 đề xuất 35.000 nhưng trả 20.000 do approval; T14 đề xuất 150.000 nhưng trả 120.000 do đã trả 30.000 trước đó. Host tự cap; không trừ lịch sử ở shape.
- Kiểm chứng refund và audit committed cùng transaction; retry sau crash vẫn một effect. Checkpoint đơn độc không bảo đảm điều này, như demo tạo 2 dòng hoàn tiền.
- Thêm `src/scripts/evaluate_swarm_linh.py` để tái lập baseline, kiểm tra an toàn public và ablation bỏ duy nhất `complete(purpose='peer')`. Patch chỉ tồn tại trong process; khôi phục hàm và registry sau thí nghiệm, giữ trace hai nhánh ở hai thư mục khác nhau.
- Cập nhật các báo cáo có sẵn. Không sửa A/B/C, Swarm tham chiếu, host, evaluator, cấu hình hoặc fixture public.


## 4. Bằng chứng

Môi trường đã chạy: Windows AMD64, Python 3.11.9, pytest 8.4.2, PyYAML 6.0.3. Dùng Python trong `.venv` ở root repo. Các lệnh dưới chạy trong `src/`; nếu chưa activate, thay `python` bằng `..\.venv\Scripts\python.exe` trong PowerShell.

| Lệnh đã chạy | Kết quả thực tế |
| --- | --- |
| `python -m lab8.smoke` | `lab8.smoke pass` |
| `python -m pytest -q` trước bổ sung test | 54 passed, 248 deselected |
| `python -m pytest -q tests/test_linh_ledger_swarm.py` | 18 passed |
| `python -m pytest -q` sau bổ sung test | 72 passed, 248 deselected |
| `python -m pytest -q -m student -o addopts= -k swarm` | 62 passed, 240 deselected ở thời điểm chạy trước khi thêm 18 test |
| `python -m lab8.demo.buggy_workflow` | `unsafe refund rows: 2` — phản ví dụ cố ý lỗi |
| `python -m lab8.crash_test --count 20 --out reports/crash.json` | 20/20 |
| `python -m lab8.benchmark --shapes swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark_swarm.json` | Baseline 69/70 |
| `python -m scripts.evaluate_swarm_linh` | Sinh lại cùng file benchmark với baseline, audit public và ablation |
| `python -m lab8.check_submission` | 6 FAIL, 1 WARN: 3 shape stub, thiếu benchmark bốn shape, chưa chọn shape, chưa có commit hash; cảnh báo chưa commit |

Đã chạy lại harness lần hai: SHA256 của toàn bộ `benchmark_swarm.json` trước/sau trùng nhau. Đây là tái lập trên cùng môi trường, chưa thay thế kiểm chứng từ commit sạch hoặc trên máy chấm. `git diff --check` không có lỗi; đối chiếu với HEAD xác nhận `src/lab8/`, `src/config/`, `src/data/`, `src/harness_map.yaml`, `src/grader_audit/` không thay đổi.

Số liệu từ `src/reports/benchmark_swarm.json` (khóa `by_shape.swarm`; ablation ở `linh_ablation.report.by_shape.swarm`):

| Metric | Swarm gốc | Bỏ peer |
| --- | ---: | ---: |
| Trial pass | 69/70 | 69/70 |
| Pass rate | 0,985714 | 0,985714 |
| Pass^5 | 0,928571 (13/14) | 0,928571 (13/14) |
| Seed min–max | 0,928571–1 | 0,928571–1 |
| Seed variance | 0,000816327 | 0,000816327 |
| Cost/success, VND mô phỏng | 49,566957 | 48,691594 |
| Tổng cost, VND mô phỏng | 3420,12 | 3359,72 |
| Mean/p95 steps (event trace) | 2,8 / 3 | 2,8 / 3 |
| Mean model/tool calls | 3 / 4,657143 | 2 / 4,657143 |

Lỗi duy nhất là `R-swarm-C-002-s55`: ticket nói trễ 5 ngày, order ghi 2 ngày; `REFERENCE_WEAKNESS` ghi đè dữ kiện ở seed 55 nên hoàn 20.000 thay vì từ chối. Seed 11/22/33/44 đều 14/14; seed 55 là 13/14. Test student của Swarm chủ động coi lỗi đối chứng này là kết quả mong đợi, nên 62 test pass không có nghĩa benchmark pass 100%.

Run id đối chứng khác: `R-swarm-C-001-s11`, `R-swarm-C-014-s11`. Trace baseline ở `src/traces/`; ablation ở `src/traces/linh_no_peer/`, cùng định dạng run id nên phải kèm đường dẫn để phân biệt. Không commit trace hoặc SQLite.

Audit local trong JSON quét 70 trial mỗi nhánh: 0 secret leak, 0 lộ giá trị nhạy cảm public, 0 approval bypass, 0 ledger ngoài claim, 0 trace sai, 0 trial chưa audit. Phạm vi gồm RunResult, trace và file cùng prefix run id, stdout/stderr bắt trong process. Không thay thế hidden grade hay chứng minh an toàn cho mọi dữ liệu.

Kết quả kiểm tra sau tích hợp do Codex thực hiện: toàn bộ pytest 419 passed, 6 xfailed; benchmark chuẩn bốn shape đã chạy lại, crash 20/20. Sáu xfail thuộc bonus B7, không phải test ledger của Linh. Báo cáo hidden_selfcheck trên main có 600 trial không nhãn của ba shape, không lỗi thực thi/trace/cảnh báo canary; không suy ra hidden pass rate. Nhóm chọn workflow trong ADR.

## 5. Công cụ AI đã dùng

Codex được dùng để đọc repo, viết test/harness, chạy lệnh và soạn nội dung dựa trên kết quả thật. Các lệnh và kết quả do Codex thực thi được ghi ở mục 4, không tự nhận là Linh đã chạy tay. Research card dẫn nguồn gốc trên ACL Anthology; giới hạn kết luận được ghi rõ.

Trước khi nộp, Linh cần tự đọc test/trace và giải thích: quote khác effect thế nào; vì sao ledger/audit cần transaction; vì sao lỗi T02 là lỗi dữ kiện chứ không phải bypass approval; vì sao bỏ peer không sửa lỗi đó. Không khai đã tự kiểm tra bằng tay khi chưa làm.

## 6. Tự đánh giá

Trạng thái kỹ thuật: đã có commit test/harness trên nhánh nộp, bằng chứng crash, benchmark Swarm và ablation. A/B/C đã tích hợp, nhóm đã chọn workflow dựa trên public benchmark; chưa có điểm hidden chính thức. Các số test/checker cũ ở bảng quá trình được giữ để mô tả thời điểm làm việc độc lập.

Phần tự đánh giá năng lực, mức độ hiểu và đóng góp thực tế: **Linh tự bổ sung sau khi đọc, chạy lại và kiểm tra nội dung**; không tự chấm điểm thay Linh.
