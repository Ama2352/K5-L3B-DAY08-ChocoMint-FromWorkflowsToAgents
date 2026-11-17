# Báo cáo nhóm — ChocoMint — Lab 8 LogiX-VN

## 1. Thông tin nhóm

- Lớp chính thức: **L3B**, nhóm trưởng đã xác nhận; mẫu SUBMISSION ghi L3A.
- Năm thành viên: Sang (tích hợp, UI, báo cáo); Nhung (workflow, crash, self-check); Quỳnh Anh (single, B7); Thảo Quyên (orchestrator, handoff, R2); Khánh Linh (swarm đối chứng, ledger, crash, benchmark).
- Nhánh nộp `main`; snapshot mã nguồn đối chiếu `f526841f182ecec31305ddf6db0750026f6d440a`, trùng ADR. Hồ sơ cập nhật bằng commit sau snapshot này.
- Repo public: https://github.com/Ama2352/K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents

## 2. Mục tiêu và cách tiếp cận

Chọn kiến trúc xử lý khiếu nại/hoàn tiền đạt chất lượng và an toàn với chi phí mô phỏng thấp. Nhóm triển khai workflow, single và orchestrator, so với swarm tham chiếu. CP1 đọc contract/cài môi trường; CP2 viết shape; CP3 retry/crash và handoff; CP4 kiểm safety; CP5 benchmark, nghiên cứu R2 và ADR.

Order và policy hiện hành tại `2026-11-15` là nguồn sự thật, không phải chữ trong ticket. Code tính tiền; host kiểm approval, trần theo order và ghi refund. Model không tạo approval.

## 3. Giải pháp

| Shape | Điều phối và ranh giới |
| --- | --- |
| Workflow | Luồng Python cố định: claim view → order theo principal → quote → host. Không gọi model. |
| Single | ReAct tối đa 8 vòng, chỉ tool đọc; giữ kết quả trong bộ nhớ. Code chọn hành động, MockLLM gọi theo bước; host ghi sau vòng lặp. |
| Orchestrator | Plan/draft qua MockLLM; investigator nhận handoff allowlist và ba tool đọc: lookup_order, lookup_policy, compute_compensation. |
| Swarm | Giữ nguyên tham chiếu làm đối chứng; điểm yếu cố ý ở C-002 seed 55. |

Shape truyền **quote đầy đủ** vào `issue_refund`; host tự trừ lịch sử trong transaction. T14 entitlement 150.000đ, đã hoàn 30.000đ → chỉ ghi thêm 120.000đ. Quote = 0 thì shape từ chối; quote > 0 mà host chặn thì có audit `policy_denied`. Approval phải đúng claim, còn hiệu lực, chưa thu hồi và đủ mức cho số tiền ghi.

Retry dùng `refund:{claim_id}`; ledger/audit nhận diện effect đã commit khi crash sau effect trước checkpoint. Truyền `crash_at`, không nuốt `InjectedCrash`; mọi return qua `finish()`. Trace đúng run_id ở mọi nhánh, không ghi raw claim/order, ticket, secret, header hoặc PII. Handoff dùng allowlist; investigator không có tool ghi.

UI tĩnh đọc JSON thật hoặc mẫu có nhãn; biểu đồ, shortlist và đối chiếu claim/seed trình bày kết quả đã chạy. Đổi lựa chọn không thực thi claim mới. Steps là event trace, không phải vòng ReAct hay latency.

## 4. Kết quả và quyết định

Benchmark chuẩn đã chạy lại: 14 claim × 5 seed `11,22,33,44,55`, cùng MockLLM, giá, policy, evaluator; mỗi trial có SQLite mới. Nguồn `src/reports/benchmark.json`, tổng 280 trial.

| Shape | Đạt | Pass rate | Pass⁵ | Seed min–max | Variance | VND/thành công | Bước TB/p95 |
| --- | ---: | ---: | ---: | --- | ---: | ---: | --- |
| workflow | 70/70 | 100% | 100% | 100–100% | 0 | 34,29 | 4,43 / 5 |
| single | 70/70 | 100% | 100% | 100–100% | 0 | 39,41 | 8,21 / 9 |
| orchestrator | 70/70 | 100% | 100% | 100–100% | 0 | 47,72 | 4,71 / 5 |
| swarm | 69/70 | 98,57% | 92,86% | 92,86–100% | 0,000816 | 49,57 | 2,80 / 3 |

Chọn **workflow**: chất lượng ngang single/orchestrator, variance 0, cost/success thấp nhất; tiết kiệm khoảng 13% so single, 28,2% so orchestrator. Cost/success tính cả chi phí trial fail; VND mô phỏng không phải hóa đơn API. Latency unavailable.

`R-swarm-C-002-s55`: ticket nói trễ 5 ngày, order ghi 2; swarm resolved thay vì denied. Workflow từ chối đúng. Research R2 phân loại lỗi kiểm chứng dữ kiện; so orchestrator/swarm còn nhiễu số model call, không quy mọi chênh lệch cho topology.

Kiểm thử: **409 passed, 6 xfailed** toàn bộ pytest; **9 test UI pass**. Sáu xfail strict là lỗi grader cố ý của B7, không phải lỗi shape. Crash suite chạy lại **20/20**: suite chung host và một ca swarm, không phải 20 ca riêng mỗi shape. T11/T12 và test bổ sung kiểm từng shape.

`hidden_selfcheck.json` từ PR #14 có **600 trial**: mỗi shape nhóm 40 claim × 5 seed; mọi reason unlabeled, không lỗi thực thi/trace/cảnh báo canary. Không có nhãn nên không tính pass rate, approval bypass hay safety hidden chính thức; chờ giảng viên.

## 5. Khó khăn và xử lý

| Vấn đề | Cách xử lý |
| --- | --- |
| UI 404/MIME module trên Windows | Chạy server từ root với --directory src; module trình duyệt dùng .js. |
| Nhầm chi phí với refund/nghìn đồng | Đối chiếu pricing: 2.400 / 70 = 34,29 VND mô phỏng/thành công. |
| Steps khác vòng ReAct | Giữ metric event trace; 8,21 event TB không có nghĩa vượt 8 vòng. |
| Conflict TEAM khi merge PR mới | Giữ cả Sang và Linh; tích hợp nguyên test/harness Linh từ 268d137 tại f526841, giữ tác giả. |
| Tài liệu có B7, checker chỉ nhận B1–B6 | Đã tái hiện 1 FAIL ở tên bonus_report.md; giữ bản B7 ở bonus_report_pending.md, chờ coach. Không sửa checker/thước đo hoặc giả topic. |

## 6. Đóng góp và hồ sơ

TEAM có năm dòng đóng góp, năm báo cáo cá nhân. Sang: `5022b6f`, `9705de5`, tích hợp #12–#15; Nhung: `ed62c93`, `0c141e9`; Quỳnh Anh: `627a70f`, `3a0a638`, `176a26b`, #14–#15; Thảo Quyên: `e95d016`, `a9450b5`, `2c8190a`; Linh: nguồn `268d137`, tích hợp `f526841`, báo cáo #12. Chi tiết theo báo cáo từng người; không gán tác giả chỉ từ JSON.

Sang xác nhận đã nộp link LMS; bốn thành viên còn lại chưa xác nhận riêng. Mỗi người tự nộp cùng link và xác nhận phần AI/tự đánh giá. Bonus đang chờ; điều kiện nhận điểm gồm hidden ≥ 0,80 và ba must-pass.

## 7. Giới hạn và điều kiện đổi lựa chọn

Public nhỏ, MockLLM và giá mô phỏng chưa đại diện production. Hidden bài chính cần ≥ 0,75 cùng approval bypass = 0, secret leak = 0, crash 20/20. Self-check không thay chấm hidden.

Đo lại khi quy trình thay đổi, dữ liệu mơ hồ, có nhiệm vụ độc lập hoặc model thật đem lại lợi ích đủ bù chi phí. Dùng corpus đại diện và giữ ranh giới an toàn, không đổi chỉ vì nhiều agent hơn.

## 8. Tái lập và nguồn

Chạy từ `src/`, môi trường Python của lab:

```powershell
python -m pytest -q -o addopts= -p no:cacheprovider
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
python -m lab8.crash_test --count 20 --out reports/crash.json
python -m scripts.evaluate_swarm_linh
python -m bonus.grader_audit_report --out reports/bonus_grader_audit.json
python -m lab8.check_submission
```

Nguồn: [business rules](../../docs/business_rules.md), [contracts](../../docs/contracts.md), [metrics](../../docs/metrics.md), [ADR](decision_adr.md), [research card](research_card.md), [checklist nộp](submission_audit.md), [bonus chờ](bonus_report_pending.md). Không sửa tay JSON; không commit trace, SQLite, .env hoặc student_drop.
