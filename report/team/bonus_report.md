# Bonus report: B7

Trạng thái tích hợp: hai hướng B7 của PR #14–#16 được giữ riêng script/test/JSON. PR #16 mở rộng checker tên topic B1–B7, không thay công thức metric hoặc grader. Gate hidden ≥ 0,80 và ba must-pass chưa được giảng viên xác nhận, nên chưa khai đã nhận điểm bonus.

- Topic + nguồn: B7 — kiểm toán grader; *Rethinking Evaluation of MAS Efficiency*, EMNLP 2026, slides 44, 59.
- CLAIM: Thước đo sai có thể đảo lệch kết luận so sánh shape, dù các shape không đổi.
- BOUNDARY: Ba hàm `summarize` được cung cấp và các fixture metamorphic tổng hợp; không đánh giá pass rate của hidden pack.
- HYPOTHESIS: Trial hạ tầng không được làm đổi pass rate; lặp nguyên bộ trial chỉ làm số tổng tăng tuyến tính; thêm trial của shape khác không được làm đổi thống kê workflow.
- Biến cố định / biến thay đổi: Dùng nguyên các fixture trong `src/grader_audit/fixtures/`; lần lượt thêm trial lỗi hạ tầng, nhân đôi danh sách trial, và thêm một trial `swarm` độc lập. Quan hệ mới kiểm tra tính độc lập giữa shape.
- Lệnh tái lập (chạy từ `src/`):

  ```text
  python -m bonus.grader_audit
  python -m pytest -q tests/test_bonus_grader_isolation.py tests/test_bonus_grader_audit.py
  python -m bonus.grader_audit_report --out reports/bonus_grader_audit.json
  ```

- File report: `src/reports/bonus_grader_isolation.json` sinh từ `bonus.grader_audit`; không có run id vì kiểm fixture tổng hợp. Audit permutation bổ sung ở `src/reports/bonus_grader_audit.json`, run id `B7-grader-audit-v1`, sinh từ `bonus.grader_audit_report`. Hai script không ghi đè cùng file.

## Kết quả

Số liệu được sinh bởi `bonus.grader_audit` từ các fixture, không nhập tay.

| Quan hệ | grader_a | grader_b | grader_c |
|---|---|---|---|
| Trial hạ tầng không đổi pass rate | Đạt: 0.75 → 0.75 | Đạt: 0.75 → 0.75 | Lệch: 0.75 → 0.5625; 4 trial timeout/crash bị tính là fail |
| Nhân đôi input: `pass_5` bất biến | Lệch: 0.6667 → 0 | Lệch: 0.6667 → 0 | Lệch: 0.6667 → 0 |
| Nhân đôi input: chi phí nhất quán | Lệch: tổng 283.5 → 473 (đáng ra 567); cost/success 31.50 → 26.28 | Đạt: tổng 189.5 → 379; cost/success giữ 21.06 | Đạt: tổng 189.5 → 379; cost/success giữ 21.06 |
| Thêm trial shape khác không đổi workflow | Đạt: pass rate giữ 0.75 | Đạt: pass rate giữ 0.75 | Đạt: pass rate giữ 0.75 |

Quan hệ nghiệp vụ riêng trong fixture `metamorphic_claim_delay.json` cũng đúng: trễ 2 ngày quote 0; trễ 3 ngày quote 35,000 VND.

## Kết luận

Không bản nào đạt mọi quan hệ. `grader_c` tính trial hạ tầng như thất bại. Cả ba bản đều làm `pass_5` giảm về 0 khi lặp nguyên danh sách: logic đếm số dòng theo claim không loại các bản sao, dù số seed duy nhất không đổi. Riêng `grader_a` cộng thêm chi phí theo claim nhưng phần cộng này không nhân đôi cùng dữ liệu, làm tổng và cost/success lệch. `grader_b` xử lý quan hệ hạ tầng và chi phí tốt nhất trong các kiểm tra này, nhưng vẫn kế thừa lỗi `pass_5`.

Quan hệ mới — thêm trial của shape khác không được đổi bất kỳ thống kê nào của workflow — đạt trên cả ba bản. Nó kiểm tra lỗi dùng chung mẫu số/chi phí giữa các shape; ba fixture cũ chỉ chứa một shape nên có thể bỏ sót lỗi này. Quan hệ metamorphic hữu ích khi không có đáp án tuyệt đối vì kiểm tra tính nhất quán của cách đo khi chỉ thay đổi một thuộc tính đã biết.

## Giới hạn

Kết quả chỉ áp dụng cho ba hàm và fixture hiện có; không chứng minh grader nào đúng trên mọi dữ liệu. Fixture không dùng model; kết luận về agent vẫn giới hạn bởi MockLLM. Đã có hidden self-check 600 trial không nhãn, nhưng chưa có hidden grade để xác nhận cổng ≥ 0,80. Các ca lỗi grader của audit permutation dùng xfail strict; audit isolation kiểm trực tiếp kết quả lệch dự kiến. Xem submission_audit.md cho số test bản tích hợp.
