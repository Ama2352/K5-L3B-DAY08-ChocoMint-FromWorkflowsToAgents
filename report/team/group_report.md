# Báo cáo nhóm — ChocoMint — Lab 8 LogiX-VN

## 1. Thông tin nhóm

- Nhóm: ChocoMint; lớp L3B theo `TEAM.md` hiện tại.
- Phân công: Sang — UI, tích hợp, bài nộp; Nhung — workflow; Quỳnh Anh — single; Thảo Quyên — orchestrator, handoff, nghiên cứu; Linh — ledger, crash, benchmark và báo cáo theo kế hoạch. Kế hoạch không thay thế bằng chứng đóng góp cá nhân.
- Commit mã nguồn dùng đối chiếu kết quả: `4ce28162c51e5092a4409f2a1e7db73884af0437`, trùng ADR. Đây là snapshot code shape đã tích hợp, chưa phải xác nhận mọi tài liệu, UI và báo cáo đã được commit/push trong bản nộp cuối.

## 2. Mục tiêu và cách tiếp cận

Chọn kiến trúc xử lý khiếu nại/hoàn tiền có chi phí mô phỏng thấp nhất trong các phương án đạt yêu cầu chất lượng và an toàn. Nhóm triển khai workflow, single ReAct và orchestrator, đối chiếu với swarm tham chiếu. Quyền sở hữu đơn, policy hiệu lực, approval, trần theo đơn và việc ghi tiền thuộc code của host; nội dung ticket không được thay thế dữ liệu order.

Các mốc triển khai: CP1 cài môi trường và đọc contract; CP2 hoàn thiện ba shape; CP3 bổ sung retry/crash và handoff; CP4 kiểm approval, trace và crash suite; CP5 benchmark, nghiên cứu và đề xuất kiến trúc. Kết quả được lấy từ công cụ lab, không chỉnh tay JSON.

## 3. Giải pháp

| Shape | Cách thực thi và ranh giới |
| --- | --- |
| Workflow | Luồng Python cố định: đọc claim view, tra order theo principal, tính quote từ order/policy rồi gọi host. Không gọi model trong bản hiện tại. |
| Single | `run_react`, tối đa 8 vòng quyết định; chỉ chọn tool đọc. Giữ các kết quả đọc trong bộ nhớ, lấy quote từ `compute_compensation`; host ghi refund sau vòng lặp. `decide` hiện theo trình tự code, MockLLM chưa quyết định hành động tự chủ. |
| Orchestrator | Gọi model cho plan/draft; `make_handoff` chuyển nhiệm vụ cho investigator qua ba tool đọc: `lookup_order`, `lookup_policy`, `compute_compensation`. Investigator không có tool ghi; host áp approval và ghi tiền. |
| Swarm | Giữ nguyên bản tham chiếu. Dùng làm đối chứng, có điểm yếu được lab thiết kế sẵn tại C-002, seed 55. |

Quote đầy đủ được truyền vào `issue_refund`; host trừ lịch sử hoàn tiền trong transaction để tránh trừ hai lần hoặc vượt entitlement. Retry cùng claim dùng idempotency key `refund:{claim_id}`. Ba shape truyền `crash_at`; có thể ghi trace khi bắt `InjectedCrash` rồi ném lại, không nuốt lỗi để giả thành công.
### Shape B — single (ReAct, tối đa 8 bước)

Shape `single` chạy một vòng ReAct bằng `lab8.runtime.run_react` với `max_steps=8`. Ở mỗi bước, agent chọn một tool đọc theo thứ tự: `get_claim_view` → `lookup_order` → `lookup_refund_history` → `compute_compensation`, rồi trả kết quả cuối. Thực tế một claim chỉ cần tối đa 5 bước, nên không bao giờ chạm trần 8.

**Ranh giới code / model.** Agent chỉ được đọc, không có tool ghi. Số tiền do `compute_compensation` tính từ dòng order trong database, không lấy từ chữ trong ticket, nên ticket nói "trễ 5 ngày" mà order ghi 2 ngày thì vẫn từ chối (T02 ở seed 55, chỗ swarm sai). MockLLM được gọi mỗi bước chỉ để tính chi phí, không quyết định số tiền hay trạng thái. Approval, trần theo đơn và idempotency đều do host kiểm.

**Gọi host.** Sau vòng lặp, code của shape mới quyết:
- Thiếu `order_id` → `needs_info`. Không thấy đơn hoặc principal không sở hữu đơn → `escalated`. Claim không tồn tại → `failed`.
- Quote bằng 0 → `denied`, không gọi host, nên không có audit `policy_denied`.
- Quote > 0 → gọi `durability.issue_refund` với **quote đầy đủ**, không tự trừ phần đã hoàn; host tự trừ (T13, T14). `crash_at` được truyền thẳng vào, `InjectedCrash` không bị bắt mà chỉ ghi trace rồi ném tiếp.
- Mọi nhánh đều trả kết quả qua `common.finish()`, nên luôn có trace đúng contract.

**Trace.** Ghi các event `react_step` (số bước và tên tool), `order` (order đã lọc field qua `lookup_order`), `refund_history` (chỉ số tiền đã hoàn), `quote` và `decision`. Cố ý **không** ghi ticket, `principal_id`, dòng claim hay order gốc, vì claim row có `parent_secret` còn order row có SĐT, địa chỉ, ghi chú nội bộ và tên hàng.

**Kết quả trên bộ public** (`src/reports/benchmark.json`, 14 claim × 5 seed): pass rate 1.000, pass^5 1.000, variance 0, cost/success 39.41 VND (mô phỏng), mean steps 8.21, p95 steps 9. "Steps" ở đây là số event trong trace, không phải số bước ReAct. Workflow (34.29 VND) rẻ hơn single khoảng 13% vì single gọi MockLLM ở mỗi bước, trong khi độ đúng của hai shape như nhau.

Handoff dùng allowlist và giới hạn tool đọc; không chuyển parent secret, header hay lịch sử thô. Trace ghi các event cần giải thích như claim view đã lọc, quote, quyết định, ReAct step hoặc handoff đã sanitize; không ghi raw claim/order, ticket hay PII. Kết quả đi qua `finish()` để tạo trace đúng contract.

UI tại `src/ui/` đọc benchmark/crash JSON, hiển thị biểu đồ cùng thang đo, đề xuất kinh tế theo ngưỡng pass rate và đối chiếu claim/seed. Nguồn báo cáo thật và mẫu được phân biệt. UI hiện xem kết quả đã chạy; không gọi shape trực tiếp.

## 4. Kết quả và quyết định

Nguồn: `src/reports/benchmark.json`. Mỗi shape chạy 14 claim × 5 seed (`11,22,33,44,55`) = 70 trial; tổng 280 trial. Cùng MockLLM, `as_of=2026-11-15`, giá và evaluator; mỗi trial dùng SQLite mới.

| Shape | Đạt | Pass rate | Pass⁵ | Seed min–max | Variance | VND/thành công | Bước TB |
| --- | ---: | ---: | ---: | --- | ---: | ---: | ---: |
| workflow | 70/70 | 100% | 100% | 100–100% | 0 | 34,29 | 4,43 |
| single | 70/70 | 100% | 100% | 100–100% | 0 | 39,41 | 8,21 |
| orchestrator | 70/70 | 100% | 100% | 100–100% | 0 | 47,72 | 4,71 |
| swarm | 69/70 | 98,57% | 92,86% | 92,86–100% | 0,000816 | 49,57 | 2,80 |

P95 steps lần lượt là 5; 9; 5; 3. Bước là số event trace, không phải latency. Tổng chi phí lần lượt là 2.400; 2.758,90; 3.340,50; 3.420,12 VND mô phỏng. Cost/success tính cả chi phí trial fail; swarm chia cho 69 lượt đạt, không phải 70.

Đề xuất chọn **workflow** cho phạm vi bài toán public: cùng chất lượng 100% với single/orchestrator, variance 0 và chi phí/thành công thấp nhất. Tiết kiệm khoảng 13,0% so với single, 28,2% so với orchestrator và 30,8% so với swarm, tính từ số gốc trước làm tròn. Không kết luận workflow luôn tốt nhất cho mọi doanh nghiệp. Chi tiết tại [decision_adr.md](decision_adr.md).

Tình huống giải thích: `C-002`, seed `55`. Ticket ghi trễ 5 ngày nhưng order là 2 ngày, nên đúng phải `denied`. Ba shape nhóm làm đúng; swarm trả `resolved`, lý do fail `status resolved != denied`, run `R-swarm-C-002-s55`. Nghiên cứu R2 xác nhận lỗi kiểm chứng dữ liệu trong phép thử này; so sánh orchestrator/swarm còn nhiễu số model call nên không quy toàn bộ chênh cost cho topology.

Kiểm tra tích hợp: smoke pass; toàn bộ pytest (bỏ bộ lọc mặc định) **363 passed**; test UI **9 passed**. Crash report **20/20**: 17 ca ở ba giai đoạn crash, hai ca conflict/approval khi retry và một ca swarm crash/resume. Đây là suite chung của host kèm ca swarm, không phải 20 ca riêng cho mỗi shape. T11/T12 và test shape bổ sung kiểm retry/crash ở các shape.

Chưa có `student_drop/` hay `hidden_grade.json`; chưa báo kết quả hidden, `secret_leak_count` hoặc `approval_bypass_count` hidden. Test public đã kiểm các ràng buộc trace, approval và ledger; public pass không chứng nhận hidden pass.

## 5. Khó khăn và cách xử lý

| Vấn đề đã gặp | Cách xử lý / bằng chứng |
| --- | --- |
| Server UI trả 404 cả `/ui/` và JS | `--directory src` phụ thuộc thư mục khởi động. Chạy từ root hoặc truyền đường dẫn tuyệt đối tới `src/`; kiểm tra lại các URL trả HTTP 200. |
| Python trên Windows phục vụ `.mjs` dạng `text/plain` | Đổi module trình duyệt thành `.js`, xác nhận `application/javascript`. |
| Dễ hiểu 34,29 VND là 34.290 VND hoặc là tổng chi phí | Đối chiếu `pricing.yaml`: giá mô phỏng theo VND; workflow 2.400/70 = 34,29 VND/thành công. UI ghi rõ đơn vị và tính mô phỏng. |
| Đếm bước không đồng nhất với số vòng ReAct | Giữ metric của lab: số event trace. Không dùng 8,21 bước TB để kết luận agent vượt giới hạn 8 vòng. |

## 6. Tổng hợp đóng góp

Sang tích hợp main vào nhánh Sang, xây UI đọc báo cáo, bộ so sánh/đề xuất và test UI; hoàn thiện bản thảo báo cáo nhóm, ADR và kịch bản demo. Nhung triển khai workflow theo phân công. Quỳnh Anh triển khai single và test riêng. Thảo Quyên triển khai orchestrator, 57 test bổ sung và research card R2. Phần việc ledger/crash/benchmark được phân cho Linh; các file kết quả hiện có được sinh khi kiểm tra tích hợp, không đủ cơ sở gán tác giả cá nhân chỉ từ JSON.

Bằng chứng commit và test cá nhân xem `TEAM.md` và các báo cáo cá nhân hiện có. Trước nộp cần bổ sung thông tin/đóng góp còn thiếu của Nhung, Linh, cập nhật commit của Sang và báo cáo cá nhân tương ứng; không coi bảng phân công là xác nhận tất cả thành viên đã hoàn tất hồ sơ.

## 7. Hạn chế và hướng cải thiện

MockLLM và bộ public nhỏ không đại diện production; latency `unavailable`; chi phí không gồm hạ tầng, vận hành, phát triển hay lỗi nghiệp vụ ngoài phép thử. Single hiện chọn hành động bằng code; không xem kết quả này là thử nghiệm suy luận tự chủ của LLM thật. Benchmark toàn shape thay đổi cả cách điều phối và số call, không phải thí nghiệm nhân quả chỉ thay topology.

Quyết định có thể đổi khi quy trình biến động, cần đọc hiểu ticket mơ hồ, công việc phân rã độc lập, workflow giảm chất lượng hoặc model thật tạo lợi ích vượt chi phí. Cần thêm corpus đại diện, hidden, đo latency/cost thật và so sánh chất lượng trong cùng ranh giới an toàn. ADR dùng ngưỡng 100% public để chọn trong ba phương án đồng chất lượng; không thay ngưỡng hidden ≥ 0,75 của lab.

## 8. Tài liệu và khả năng chạy lại

- [Luật nghiệp vụ](../../docs/business_rules.md), [contract](../../docs/contracts.md), [kiến trúc](../../docs/architecture.md), [metrics](../../docs/metrics.md).
- [Bảng giá mô phỏng](../../src/config/pricing.yaml); [hướng dẫn sinh báo cáo](../../src/reports/README.md).
- [Research card và giới hạn ablation R2](research_card.md); [kịch bản demo và vấn đáp](../../docs/demo_script.md).

Lệnh chạy lại trong `src/` (Python của môi trường lab):

```powershell
python -m lab8.smoke
python -m pytest -q -o addopts= -p no:cacheprovider
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
python -m lab8.crash_test --count 20 --out reports/crash.json
python -m lab8.check_submission
```

Trước nộp: commit code/test/report, xác nhận hash mã nguồn và run ID trong ADR, cập nhật hồ sơ cá nhân/TEAM, chạy checker và push. Checker đạt không đồng nghĩa đã đủ mọi thông tin cá nhân hoặc đã qua grader hidden.
