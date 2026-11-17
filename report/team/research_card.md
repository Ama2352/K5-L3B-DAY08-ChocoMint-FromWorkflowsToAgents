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
| CÂU | R1 / R2 / R3 / R4 | R2 · Failure taxonomy |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Cemri và cộng sự, "Why Do Multi-Agent LLM Systems Fail?", NeurIPS 2025, https://arxiv.org/abs/2503.13657. Loại bằng chứng: phân loại lỗi (taxonomy) xây từ trace thật của các hệ multi-agent do người gán nhãn. Tôi đã kiểm tra tiêu đề, tác giả và URL của paper. Các con số 14 failure mode, 3 nhóm và kappa 0,88 lấy từ mô tả của lab trong docs/bonus_topics.md (B2), chưa đối chiếu từng con số với bản paper |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Lỗi của hệ multi-agent LLM chia được thành 14 failure mode trong 3 nhóm: lỗi thiết kế hệ thống, lệch pha giữa các agent, và lỗi kiểm chứng kết quả (task verification) |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Taxonomy dựng từ trace của các framework multi-agent chạy model LLM thật trên benchmark của họ. Lab này dùng MockLLM cố định theo seed, 14 claim public và 5 seed, nên không có lỗi sinh văn bản tự do. Kết luận chỉ nói về việc gán nhãn lỗi, không nói multi-agent tốt hay tệ hơn nói chung |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Trial fail duy nhất của swarm (C-002, seed 55) thuộc nhóm task verification: số ngày trong ticket không được kiểm với cột delay_days của order. Nếu quote chỉ lấy từ order row như orchestrator thì pass rate trên 70 trial tăng từ khoảng 0,986 lên 1,000, pass^5 từ khoảng 0,929 lên 1,000 và variance giữa các seed về 0. Dự đoán này dựa trên điểm yếu REFERENCE_WEAKNESS đã được lab ghi sẵn, nên kết quả gần như được báo trước |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ cố định: MockLLM, ba tool đọc (lookup_order, lookup_policy, compute_compensation), handoff allowlist, host durability.issue_refund, 14 claim public, seed 11,22,33,44,55, bảng giá, evaluator, mỗi trial một SQLite mới. Biến thay đổi: nguồn của delay_days, orchestrator chỉ đọc order row, còn swarm ở seed chia hết cho 5 có thể bị ticket ghi đè. Nhiễu còn lại: swarm gọi model 3 lần mỗi trial (peer, parse, draft), orchestrator gọi 2 lần (plan, draft), nên chênh cost không quy hết cho biến trên |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: quote theo số ngày ghi trong ticket thay vì order row, dẫn tới refund 20.000 VND khi đáng ra phải denied. Đo bằng pass rate, pass^5, seed variance, cost/success, secret leak và approval bypass |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/research_ablation_r2.json` (chạy trong `src/`)
- Run id / file report: `src/reports/research_ablation_r2.json`. Trial fail duy nhất: `R-swarm-C-002-s55` (status `resolved`, nhãn đúng là `denied`)
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **đúng**, trên 70 trial mỗi shape.

  | Shape | Pass rate | pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean steps |
  | --- | --- | --- | --- | --- | --- | --- |
  | orchestrator | 1,000 | 1,000 | 1,000–1,000 | 0 | 47,72 | 4,71 |
  | swarm | 0,986 | 0,929 | 0,929–1,000 | 0,000816 | 49,57 | 2,80 |

  Mọi lỗi nằm ở C-002 seed 55: ticket ghi "trễ 5 ngày", order ghi 2 ngày, swarm tin ticket nên trả 20.000 VND. Đây là lỗi nhóm task verification. Orchestrator đọc `delay_days` từ order row nên không mắc lỗi này. Orchestrator gọi model ít hơn (2 so với 3 lần mỗi trial) nhưng có nhiều event trace hơn (4,71 so với 2,80 bước), nên chênh cost/success chỉ là gợi ý, không quy được riêng cho biến đã đổi.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. Chỉ có một trial fail nên không đủ để khái quát về taxonomy. Hypothesis dựa trên điểm yếu lab đã ghi sẵn, nên kết quả xác nhận chứ không phát hiện mới. Điều kiện làm kết luận đảo lại: nếu order row sai hoặc lệch so với thực tế thì việc bỏ qua ticket sẽ gây lỗi ngược lại.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
