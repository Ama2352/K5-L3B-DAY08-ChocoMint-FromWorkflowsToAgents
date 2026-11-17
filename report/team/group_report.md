# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm:
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`):

## 2. Mục tiêu và cách tiếp cận

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

...

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

### Shape B — single (ReAct, tối đa 8 bước)

Shape `single` chạy một vòng ReAct bằng `lab8.runtime.run_react` với `max_steps=8`. Ở mỗi bước, agent chọn một tool đọc theo thứ tự: `get_claim_view` → `lookup_order` → `lookup_refund_history` → `compute_compensation`, rồi trả kết quả cuối. Thực tế một claim chỉ cần tối đa 5 bước, nên không bao giờ chạm trần 8.

**Ranh giới code / model.** Agent chỉ được đọc, không có tool ghi. Số tiền do `compute_compensation` tính từ dòng order trong database, không lấy từ chữ trong ticket, nên ticket nói "trễ 5 ngày" mà order ghi 2 ngày thì vẫn từ chối (T02 ở seed 55, chỗ swarm sai). MockLLM được gọi mỗi bước chỉ để tính chi phí, không quyết định số tiền hay trạng thái. Approval, trần theo đơn và idempotency đều do host kiểm.

**Gọi host.** Sau vòng lặp, code của shape mới quyết:
- Thiếu `order_id` → `needs_info`. Không thấy đơn hoặc principal không sở hữu đơn → `escalated`. Claim không tồn tại → `failed`.
- Quote bằng 0 → `denied`, không gọi host, nên không có audit `policy_denied`.
- Quote > 0 → gọi `durability.issue_refund` với **quote đầy đủ**, không tự trừ phần đã hoàn; host tự trừ (T13, T14). `crash_at` được truyền thẳng vào, `InjectedCrash` không bị bắt mà chỉ ghi trace rồi ném tiếp.
- Mọi nhánh đều trả kết quả qua `common.finish()`, nên luôn có trace đúng contract.

**Trace.** Ghi các event `react_step` (số bước và tên tool), `order` (order đã lọc field qua `lookup_order`), `refund_history` (chỉ số tiền đã hoàn), `quote` và `decision`. Cố ý **không** ghi ticket, `principal_id`, dòng claim hay order gốc, vì claim row có `parent_secret` còn order row có SĐT, địa chỉ, ghi chú nội bộ và tên hàng.

**Kết quả trên bộ public** (`src/reports/benchmark.json`, 14 claim × 5 seed): pass rate 1.000, pass^5 1.000, variance 0, cost/success 39.41 VND (mô phỏng), mean steps 8.21, p95 steps 9. "Steps" ở đây là số event trong trace, không phải số bước ReAct. So với workflow (34.29 VND), single đắt hơn khoảng 15% vì gọi MockLLM ở mỗi bước, nhưng không đúng hơn.

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

...

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

...

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
