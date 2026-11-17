# Bonus report: B7

- Topic + nguồn: B7 — kiểm toán grader; *Rethinking Evaluation of MAS Efficiency*, EMNLP 2026, slides 44, 59.
- CLAIM: Thước đo sai có thể đảo lệch kết luận so sánh shape, dù các shape không đổi.
- BOUNDARY: Ba hàm `summarize` được cung cấp và các fixture metamorphic tổng hợp; không đánh giá pass rate của hidden pack.
- HYPOTHESIS: Trial hạ tầng không được làm đổi pass rate; lặp nguyên bộ trial chỉ làm số tổng tăng tuyến tính; thêm trial của shape khác không được làm đổi thống kê workflow.
- Biến cố định / biến thay đổi: Dùng nguyên các fixture trong `src/grader_audit/fixtures/`; lần lượt thêm trial lỗi hạ tầng, nhân đôi danh sách trial, và thêm một trial `swarm` độc lập. Quan hệ mới kiểm tra tính độc lập giữa shape.
- Lệnh tái lập (chạy từ `src/`):

  ```text
  python -m bonus.grader_audit
  python -m pytest -q tests/test_bonus_grader_audit.py
  ```

- File report + run id: `src/reports/bonus_grader_audit.json`; không có run id vì đây là kiểm toán fixture tổng hợp, không chạy một trial shape.

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

Kết quả chỉ áp dụng cho ba hàm và fixture hiện có; không chứng minh grader nào đúng trên mọi dữ liệu. Fixture không dùng model; các kết luận về hành vi agent thực tế vẫn bị giới hạn bởi MockLLM. Bộ hidden không có trong workspace, nên chưa xác nhận được cổng pass rate hidden ≥ 0.80. Từ `src/`, `python -m pytest -q` đạt 67 passed, 248 deselected.
