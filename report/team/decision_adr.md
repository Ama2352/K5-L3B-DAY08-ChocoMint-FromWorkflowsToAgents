# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn:
- Commit hash:
- Run id dẫn chứng: Đối chứng `R-swarm-C-002-s55`, `R-swarm-C-001-s11`, `R-swarm-C-014-s11`; xem `src/reports/benchmark_swarm.json`. Chưa có run id của shape được nhóm chọn.
- Pass rate / pass^5 / seed min-max / variance: Swarm public 69/70 = 0,985714 / 13/14 = 0,928571 / 0,928571–1 / 0,000816327. Chưa phải kết quả so bốn shape hoặc hidden.
- Cost per success (mô phỏng, VND): Swarm gốc 49,566957; ablation bỏ peer 48,691594, cùng 70 trial và 5 seed.
- Crash safety: 20/20 (`src/reports/crash.json`, host + Swarm); shape nhóm chọn còn chờ T12 sau merge.
- Secret leak / approval bypass: Audit local public của Swarm: 0 / 0 trên 70 trial; quét giá trị nhạy cảm cũng 0. Xem `linh_public_safety` trong JSON; chưa có hidden grade.

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

**Bản ghi tạm, chưa chọn shape.** Hai dòng `Shape chọn` và `Commit hash` cố ý để trống cho đến khi có benchmark A/B/C/D và commit thật. HEAD nền `034c6c421c198dae67a9211d15e2587d5a378972` không chứa phần việc đang sửa nên không dùng làm commit chốt. Phần Linh ở nhánh local `linh`, chưa commit/push/merge.

Swarm là đối chứng, không mặc định được chọn vì có sẵn. Nó đúng 69/70 trial public và sai T02 seed 55 do tin số ngày trong ticket. Nhóm cần chọn kiến trúc ít tự chủ nhất đạt ngưỡng và must-pass, dựa trên cùng corpus/seed/giá sau merge. Hiện chưa đủ dữ liệu kết luận A/B/C tốt hơn hay rẻ hơn D.

## Có đáng đổi lấy multi-agent không

Ablation bỏ riêng lời gọi peer giữ nguyên từng quyết định/ledger/audit và từng nhãn pass/fail, giảm tổng cost mô phỏng 60,40 VND. Lời gọi peer hiện chỉ trả acknowledgement không được downstream sử dụng; kết quả này đo overhead của implementation hiện tại, không chứng minh multi-agent luôn kém hơn.

Sau merge, so pass rate, pass^5, variance và cost/success của bốn shape. Chỉ đổi sang kiến trúc phức tạp hơn nếu lợi ích đo được đủ bù chi phí và vẫn qua safety gates. Kết luận phải xem lại nếu task cần suy luận mơ hồ, các investigator độc lập thực sự, hoặc model thật cho kết quả khác.

## Trace và bằng chứng

Swarm ghi handoff đã sanitize, investigator với order view được phép và số policy, quote; thêm decision/error/crash/model_override tùy nhánh. Không ghi parent secret, header, điện thoại, địa chỉ, internal note, item name hoặc raw claim/order. Handoff chỉ cấp read tools; host là writer.

Script `python -m scripts.evaluate_swarm_linh` quét kết quả, trace cùng prefix run id và stdout/stderr trên toàn bộ public. Trace ablation ở `src/traces/linh_no_peer/`, baseline ở `src/traces/`; run id giống nhau nên cần kèm thư mục. Không có bằng chứng cho file/log ngoài vùng quét; vì vậy không tuyên bố đã chứng minh mọi kênh không rò rỉ.

## Giới hạn

MockLLM không phải model sản xuất; latency là `unavailable`; giá mô phỏng, 14 claim public. Chưa có hidden grade. Crash suite 20/20 chủ yếu kiểm host và có một scenario Swarm; không thay thế T12 của A/B/C.

Sau merge, chạy từ `src/`:

1. `python -m pytest -q` và `python -m pytest -q -m student -o addopts=`.
2. `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`.
3. Cập nhật shape chọn, số liệu, run id, commit thật; commit/push/merge chỉ khi được yêu cầu.
4. `python -m lab8.check_submission`; hiện chưa thể 0 FAIL do A/B/C stub, thiếu benchmark bốn shape và chưa chốt ADR.
