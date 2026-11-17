# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R4 — Evaluation confounders; ablation một thành phần Swarm, thực hiện được trước merge A/B/C. |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Jiamu Zhang và cộng sự, 2026, [Rethinking the Evaluation of Efficiency Methods for Multi-Agent Systems](https://aclanthology.org/2026.emnlp-main.798/), EMNLP; nghiên cứu benchmark có kiểm soát. Dùng abstract/trang công bố để xác nhận claim, không nhận đã tái lập toàn bộ paper. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Kết quả về hiệu quả có thể phụ thuộc setup; cần cố định model, registry và runtime để phân biệt tác động của thay đổi cấu trúc. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Nguồn đánh giá các phương pháp hiệu quả MAS trong benchmark chẩn đoán của họ. Thí nghiệm ở đây chỉ kiểm Swarm Python/MockLLM trên 14 claim giả lập, không chuyển kết luận thành quy luật cho agent sản xuất. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Bỏ duy nhất lời gọi peer sẽ giữ nguyên từng quyết định và pass rate, giảm đúng 1 model call/trial và giảm cost/success; dự đoán không sửa lỗi T02 seed 55. Không ấn định mức giảm VND trước khi đo. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ nguyên MockLLM, prompt của parse/draft, tools, public world/claims, as_of 2026-11-15, seeds 11/22/33/44/55, pricing và evaluator; DB mới mỗi trial. Chỉ bỏ complete với purpose peer bằng patch trong process. Chạy cặp 70 trial mỗi nhánh; khôi phục hàm sau chạy; trace hai thư mục riêng. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Kiểm từng cặp status, proposal, số tiền ledger và audit, cùng nhãn pass/fail; đo model calls, cost/success, total cost, steps, pass^5 và variance. Audit public quét secret/PII, approval và ledger ngoài claim. Lỗi dự kiến còn nguyên: Swarm tin ticket ở T02 seed 55. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): từ `src/`, `..\.venv\Scripts\python.exe -m scripts.evaluate_swarm_linh`; test harness bằng `..\.venv\Scripts\python.exe -m pytest -q tests/test_linh_ledger_swarm.py`.
- Run id / file report: file có sẵn `src/reports/benchmark_swarm.json`. Baseline ở `by_shape.swarm`; ablation ở `linh_ablation.report.by_shape.swarm`. Ví dụ `R-swarm-C-002-s55`: trace baseline `src/traces/`, ablation `src/traces/linh_no_peer/`. Không tạo thêm file báo cáo.
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: ủng hộ trong setup này; `paired_business_outcomes_equal` và `paired_grades_equal` đều true. Cả hai nhánh đúng 69/70, pass^5 13/14, variance 0,000816327; mean model calls 3 → 2, cost/success 49,566957 → 48,691594 VND mô phỏng. Tổng cost 3420,12 → 3359,72, giảm 60,40 VND. Mean/p95 steps vẫn 2,8/3 vì metric steps đếm event trace, không đếm mọi model call. Hai nhánh đều 0 leak/bypass theo audit local trên 70 trial.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Lời gọi peer gốc trả acknowledgement rồi bị bỏ qua; ablation này đo overhead có thể dự đoán từ code, không phải bằng chứng mới rằng mọi hình thức cộng tác đều vô ích. Nó không thay topology investigator, không sửa `REFERENCE_WEAKNESS` và không thay thế so sánh A/B/C/D. Kết luận có thể thay đổi khi peer output thực sự tham gia quyết định hoặc task khác cần nhiều nguồn suy luận. Đây là research card của bài chính, chưa đăng ký hoặc nhận điểm bonus B4.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
