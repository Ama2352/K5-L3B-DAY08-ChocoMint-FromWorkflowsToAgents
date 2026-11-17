# Bonus report: B7

- Topic + nguồn (tên, năm, slide): B7 · Kiểm toán grader. *Rethinking Evaluation of MAS Efficiency*, EMNLP 2026, slide 44 và 59.
- CLAIM của nguồn (1 câu): Kết luận về hiệu năng của hệ multi-agent có thể đến từ cách đo, không phải từ hệ thống được đo.
- BOUNDARY (model / dataset / setup): Paper xét các benchmark MAS dùng LLM thật. Ở đây không có model thật; đối tượng bị kiểm là ba bản hàm `summarize` (`grader_audit/grader_a.py`, `grader_b.py`, `grader_c.py`) chạy trên các fixture nhỏ do giảng viên cung cấp.
- HYPOTHESIS trên LogiX: Ít nhất một trong ba bản grader làm lệch số liệu mà không có test nào báo. Áp các quan hệ metamorphic sẽ chỉ ra bản nào lệch và lệch ở số nào.
- Biến cố định / biến thay đổi: Cố định fixture, `lab8.metrics.summarize` gốc và luật nghiệp vụ. Chỉ thay đổi bản grader (A, B, C) và phép biến đổi input theo từng quan hệ.
- Lệnh tái lập (copy-paste được, chạy trong `src/`):
  - `python -m pytest -q tests/test_bonus_grader_audit.py`
  - `python -m bonus.grader_audit_report --out reports/bonus_grader_audit.json`
- File report + run id: `src/reports/bonus_grader_audit.json`, run id `B7-grader-audit-v1`.

## Kết quả

Bảng lấy từ `src/reports/bonus_grader_audit.json`. "Lệch" kèm số trước → sau và số đúng ra phải là.

| Quan hệ | grader_a | grader_b | grader_c |
| --- | --- | --- | --- |
| Thêm trial lỗi hạ tầng không làm đổi pass rate | Đạt | Đạt | **Lệch**: pass_rate 0.75 → 0.5625 (phải giữ 0.75) |
| Lặp danh sách: tỉ lệ giữ nguyên, tổng gấp đôi | **Lệch**: tổng chi phí 283.5 → 473 (phải 567); cost/success 31.5 → 26.28 (phải giữ 31.5); pass^5 0.667 → 0 | **Lệch**: pass^5 0.667 → 0 | **Lệch**: pass^5 0.667 → 0 |
| **Quan hệ mới:** đổi thứ tự trial thì mọi số giữ nguyên | **Lệch**: tổng chi phí 283.5 → 284.0; cost/success 31.5 → 31.56 | Đạt | Đạt |
| Trễ 2 ngày so với 3 ngày (luật nghiệp vụ, `compensation.quote`) | Đạt: 2 ngày → 0 VND, 3 ngày → 35.000 VND (bằng phí ship, đúng `business_rules.md`) | | |

Tên lỗi theo hành vi:

- **grader_c — đếm trial hạ tầng vào mẫu số.** Trial chết vì timeout hoặc crash hạ tầng bị đổi thành `passed=False`, nên bị tính như quyết định sai của shape và kéo pass rate xuống.
- **grader_a — đếm chi phí hai lần, và phụ thuộc thứ tự dòng.** Nó cộng thêm chi phí của dòng *đầu tiên gặp được* của mỗi claim vào tổng vốn đã có chi phí đó. Ngay trên danh sách gốc, cost/success đã là 31.5 thay vì 21.06. Vì cộng theo claim chứ không theo trial, nhân đôi danh sách thì tổng không gấp đôi; đổi thứ tự thì số cộng thêm đổi theo.
- **lỗi chung ở `lab8/metrics.py` — pass^5 về 0 khi có dòng lặp.** Hàm gốc chỉ tính một claim là đạt pass^5 khi nó có **đúng một dòng cho mỗi seed** (`len(claim_rows) == len(seeds)`). Cả ba bản đều gọi lại hàm này nên cùng lệch. Nhóm không sửa `metrics.py` (file thước đo), chỉ ghi nhận.

Bản đúng nhất: **grader_b** — đạt mọi quan hệ, trừ lỗi pass^5 kế thừa từ hàm gốc.

**Quan hệ mới — đổi thứ tự trial.** Ba quan hệ cũ không bao giờ đổi thứ tự dòng, nên không kiểm được grader có phụ thuộc thứ tự hay không. Một grader đúng phải cho cùng kết quả dù trial được ghi theo thứ tự nào (chạy song song, đọc từ nhiều file). Trên fixture hiện có, quan hệ này bắt được grader_a theo một hành vi khác với quan hệ lặp danh sách: lặp danh sách cho thấy A cộng sai tổng, còn đổi thứ tự cho thấy kết quả của A tùy vào dòng nào đứng trước.

## Kết luận

**Ủng hộ** hypothesis. Hai trong ba bản grader (A, C) làm lệch số liệu, và cả ba cùng lệch pass^5 khi có dòng lặp. Không có test sẵn nào trong repo báo các lỗi này (smoke test chỉ kiểm import và kiểu trả về). Nếu dùng grader_c, mọi shape hay gặp timeout sẽ bị đánh giá thấp hơn thực tế; nếu dùng grader_a, so sánh cost/success giữa các shape bị sai — đúng loại lệch mà nguồn cảnh báo.

Trả lời câu hỏi của topic:

- *Lỗi thước đo có thể lọt qua mọi quan hệ nhóm đã viết không?* Có. Các quan hệ chỉ kiểm cách output **thay đổi** khi input đổi. Một grader sai theo cách nhất quán (ví dụ luôn nhân chi phí với 1.1) vẫn thỏa cả bốn quan hệ. Còn thiếu một quan hệ dạng "nhân mọi `cost_estimate` với k thì cost/success nhân đúng k", và ít nhất một ca có đáp án tính tay để neo giá trị tuyệt đối.
- *Vì sao quan hệ metamorphic hữu ích khi không có đáp án tuyệt đối?* Với số tiền hay pass rate trên dữ liệu lớn, khó biết con số đúng là bao nhiêu. Nhưng biết chắc một số điều phải đúng: thêm trial hạ tầng không được đổi pass rate, lặp dữ liệu không được đổi tỉ lệ, đổi thứ tự không được đổi gì. Kiểm các ràng buộc đó không cần nhãn mà vẫn bắt được lỗi thật.

**Khi nào kết luận đảo lại:** nếu fixture không có trial hạ tầng, không có dòng lặp, hoặc mỗi claim có cùng chi phí ở mọi seed, thì cả ba bản cho cùng kết quả và không phân biệt được.

## Giới hạn

- Không đo shape nào; MockLLM và giá mô phỏng không ảnh hưởng kết quả, vì đối tượng là grader chứ không phải agent.
- Fixture nhỏ (12–24 trial, một shape `workflow`). Lỗi chỉ hiện khi dữ liệu có đúng đặc điểm gây lỗi.
- Quan hệ mới chỉ bắt lại grader_a, bản đã bị quan hệ lặp danh sách bắt; chưa có fixture riêng cho thấy một lỗi chỉ quan hệ mới bắt được.
- Các ca lệch đã biết được đánh dấu `xfail(strict=True)` để `python -m pytest -q` vẫn xanh; nếu bản grader thay đổi và hết lệch, test sẽ báo đỏ.
- Bonus chỉ được tính khi shape nhóm chọn (`workflow`) đạt hidden ≥ 0.80 và qua 3 must-pass; nhóm chưa có `hidden_grade.json` để xác nhận.
