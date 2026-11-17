# Báo cáo cá nhân — 2A202603875_VuNgocQuynhAnh

## 1. Thông tin thành viên

- Họ tên: Vũ Ngọc Quỳnh Anh
- MSSV: 2A202603875
- Email: 26ai.anhvnq@vinuni.edu.vn
- GitHub: qavu2407-lang

## 2. Phần việc phụ trách

Tôi làm shape B `single`: một agent ReAct, tối đa 8 bước, trong file `src/lab8/shapes/single.py`.

## 3. Đóng góp cụ thể

- Commit chính: `627a70f` (code shape single), `3a0a638` (test cơ bản), `176a26b` (test approval và đơn đã hoàn)
- Test thêm mới:
  - `src/tests/test_single_react.py`: kiểm agent không quá 8 bước, chỉ gọi tool đọc, không ghi ticket vào trace, và vẫn từ chối C-002 ở seed 55 dù ticket nói trễ 5 ngày.
  - `src/tests/test_single_extra.py`: kiểm host chặn đúng khi approval hỏng (thiếu, không tồn tại, của claim khác, bị thu hồi, hết hạn, số tiền 0) và khi đơn đã được hoàn đủ (T13, hoặc claim khác vừa hoàn cùng đơn). Mọi trường hợp đều ra `denied`, giữ quote đầy đủ, không có dòng ledger và có đúng một audit `policy_denied` với lý do đúng.
- Quyết định kỹ thuật:
  - Agent chỉ được gọi tool đọc theo thứ tự: xem claim → tra đơn → tra lịch sử hoàn tiền → tính tiền. Việc ghi refund do code của shape làm sau vòng lặp, không để agent làm.
  - Agent không đọc chữ trong ticket. Số ngày trễ và số tiền lấy từ dữ liệu đơn hàng, tính bằng `compute_compensation`. MockLLM chỉ để đếm chi phí, không quyết tiền.
  - Truyền nguyên quote vào `issue_refund`, không tự trừ phần đã hoàn. Host tự trừ, nếu shape trừ trước thì sẽ bị trừ hai lần (T14).
  - Quote bằng 0 thì shape tự trả `denied` và không gọi host, nên không có audit `policy_denied`. Quote > 0 mà host từ chối thì mới có audit này.
  - Mọi nhánh trả kết quả đều đi qua `finish()` nên luôn có trace. Khi crash thì ghi trace rồi ném lỗi tiếp, không bắt lại.
  - Trace chỉ ghi tên tool, order đã lọc field, số tiền đã hoàn, quote và quyết định. Không ghi ticket, `principal_id` hay dòng claim/order gốc.

## 4. Bằng chứng

- `python -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"`: 62 passed.
- `python -m pytest -q`: xanh (133 passed sau khi merge `main`), gồm 4 test trong `test_single_react.py` và 18 test trong `test_single_extra.py`.
- Chạy cả 14 claim public ở seed 55: kết quả đúng với luật nghiệp vụ. Ví dụ `R-single-C-014-s55`: `resolved`, quote 150.000, ledger 120.000. `R-single-C-013-s55`: `denied`, có `policy_denied` lý do `order_already_refunded`.
- Tìm `CANARY_SECRET` và `parent_secret` trong 65 file trace `R-single-*`: không có.

## 5. Công cụ AI đã dùng

Tôi dùng Claude Code để đọc tài liệu của lab, gợi ý code cho `single.py`, viết test và lập checklist. Tôi tự kiểm lại bằng cách:

- Đọc lại code, đối chiếu với `docs/business_rules.md` và `docs/contracts.md`.
- Chạy 62 test student của shape `single` và cổng host.
- Chạy từng claim public, xem `status`, quote, ledger và audit `policy_denied` có đúng không.
- Tìm secret trong trace.

## 6. Tự đánh giá

Shape `single` đã qua hết test public và các điều kiện an toàn. Điểm hạn chế là `decide()` đang đi theo thứ tự cố định vì MockLLM không tự suy luận được, nên shape này chưa thật sự "tự chủ" như một agent dùng model thật. Nếu có thêm thời gian, tôi sẽ bắt thêm lỗi lạ trong `run_claim` để không bao giờ ném exception ra ngoài (trừ `InjectedCrash`).
