# Bonus report: B7

Trạng thái tích hợp: hai hướng B7 của PR #14–#16 được giữ riêng script/test/JSON. PR #16 mở rộng checker tên topic B1–B7, không thay công thức metric hoặc grader. Gate hidden ≥ 0,80 và ba must-pass chưa được giảng viên xác nhận, nên chưa khai đã nhận điểm bonus.

- Topic + nguồn: B7 — kiểm toán grader; *Rethinking Evaluation of MAS Efficiency*, EMNLP 2026, slides 44, 59.
- CLAIM: Thước đo sai có thể đảo lệch kết luận so sánh shape, dù các shape không đổi.
- BOUNDARY: Ba hàm `summarize` được cung cấp và các fixture metamorphic tổng hợp; không đánh giá pass rate của hidden pack.
- HYPOTHESIS: Trial hạ tầng không làm đổi pass rate; lặp bộ trial chỉ làm số tổng tăng tuyến tính; đảo thứ tự không đổi thống kê; thêm shape khác không đổi thống kê workflow.
- Biến cố định / biến thay đổi: Giữ fixture, metric gốc và luật nghiệp vụ; lần lượt thêm lỗi hạ tầng, nhân đôi danh sách, đảo thứ tự trial hoặc thêm một trial `swarm` độc lập. Hai quan hệ mới kiểm thứ tự dòng và tính độc lập giữa shape.
- Lệnh tái lập (chạy từ `src/`):

  ```text
  python -m bonus.grader_audit
  python -m pytest -q tests/test_bonus_grader_isolation.py tests/test_bonus_grader_audit.py
  python -m bonus.grader_audit_report --out reports/bonus_grader_audit.json
  ```

- File report: `src/reports/bonus_grader_isolation.json` sinh từ `bonus.grader_audit`; không có run id vì kiểm fixture tổng hợp. Audit permutation bổ sung ở `src/reports/bonus_grader_audit.json`, run id `B7-grader-audit-v1`, sinh từ `bonus.grader_audit_report`. Hai script không ghi đè cùng file.

## Kết quả

Số liệu từ hai script/JSON nêu trên; hàng đảo thứ tự lấy từ `bonus_grader_audit.json`, các hàng còn lại từ `bonus_grader_isolation.json`. Không nhập tay JSON.

| Quan hệ | grader_a | grader_b | grader_c |
|---|---|---|---|
| Trial hạ tầng không đổi pass rate | Đạt: 0.75 → 0.75 | Đạt: 0.75 → 0.75 | Lệch: 0.75 → 0.5625; 4 trial timeout/crash bị tính là fail |
| Nhân đôi input: `pass_5` bất biến | Lệch: 0.6667 → 0 | Lệch: 0.6667 → 0 | Lệch: 0.6667 → 0 |
| Nhân đôi input: chi phí nhất quán | Lệch: tổng 283.5 → 473 (đáng ra 567); cost/success 31.50 → 26.28 | Đạt: tổng 189.5 → 379; cost/success giữ 21.06 | Đạt: tổng 189.5 → 379; cost/success giữ 21.06 |
| Thêm trial shape khác không đổi workflow | Đạt: pass rate giữ 0.75 | Đạt: pass rate giữ 0.75 | Đạt: pass rate giữ 0.75 |
| Đảo thứ tự trial không đổi thống kê | Lệch: tổng chi phí 283.5 → 284; cost/success 31.50 → 31.56 | Đạt | Đạt |

Quan hệ nghiệp vụ riêng trong fixture `metamorphic_claim_delay.json` cũng đúng: trễ 2 ngày quote 0; trễ 3 ngày quote 35,000 VND.

## Kết luận

Kết quả **ủng hộ hypothesis rằng cách đo có thể làm lệch so sánh**. Không bản nào đạt mọi quan hệ. `grader_c` đếm trial hạ tầng vào mẫu số như thất bại. Cả ba kế thừa lỗi `pass_5`: khi lặp nguyên danh sách, số dòng theo claim không còn bằng số seed nên tỷ lệ về 0. `grader_a` cộng thêm chi phí của dòng đầu mỗi claim vào tổng đã có; khoản cộng không tăng tuyến tính và phụ thuộc thứ tự dòng. `grader_b` tốt nhất trong các phép kiểm, nhưng vẫn kế thừa lỗi `pass_5`. Không sửa metric/grader do giảng viên cung cấp.

Quan hệ mới — thêm trial của shape khác không được đổi bất kỳ thống kê nào của workflow — đạt trên cả ba bản. Nó kiểm tra lỗi dùng chung mẫu số/chi phí giữa các shape; ba fixture cũ chỉ chứa một shape nên có thể bỏ sót lỗi này. Quan hệ metamorphic hữu ích khi không có đáp án tuyệt đối vì kiểm tra tính nhất quán của cách đo khi chỉ thay đổi một thuộc tính đã biết.

Quan hệ đảo thứ tự bắt tính phụ thuộc dòng đứng trước của `grader_a`; các phép thử cũ không hoán vị dòng. Nó bổ sung cách giải thích lỗi chi phí, nhưng chưa có fixture riêng chứng minh chỉ phép thử mới này bắt được lỗi. Không khẳng định hai quan hệ mới phát hiện thêm hai lỗi độc lập.

**Lỗi có thể lọt qua mọi phép thử không?** Có: grader luôn nhân mọi chi phí với hệ số sai cố định vẫn có thể thỏa các quan hệ hiện tại. Cần thêm ca có đáp án tính tay để neo giá trị tuyệt đối; phép thử nhân mọi chi phí với k cũng không tự phát hiện hệ số sai cố định đó.

**Điều kiện thay đổi kết luận:** khi sửa các grader hoặc thay dữ liệu, phải đo lại. Nếu không có lỗi hạ tầng thì phép thử tương ứng không phân biệt C; chi phí bằng nhau giữa các seed có thể che phụ thuộc thứ tự của A. Không có dòng lặp sẽ không kích hoạt lỗi pass⁵. Các điều kiện này có thể che lỗi, không chứng minh grader đã đúng.

## Giới hạn

Chỉ xét ba hàm và fixture nhỏ, không chứng minh độ đúng trên mọi dữ liệu hay hiệu quả production. Fixture không dùng model; đánh giá agent vẫn giới hạn bởi MockLLM và giá mô phỏng, latency unavailable. Hidden self-check 600 trial không nhãn chưa xác nhận gate ≥ 0,80 và safety chính thức. Audit permutation dùng xfail strict cho lỗi đã biết; isolation kiểm trực tiếp kết quả lệch dự kiến. Xem submission_audit.md cho số test tích hợp.
