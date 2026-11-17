# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 72a00f2 (HEAD lúc benchmark; worktree có thay đổi chưa commit)
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000 (70 trial public)
- Cost per success (mô phỏng, VND): 34.29
- Crash safety: 20/20 (`src/reports/crash.json`)
- Secret leak / approval bypass: Không phát hiện trên benchmark/test public; hidden chưa xác minh vì không có `student_drop/`.

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Nhóm chọn workflow vì luồng quyết định xác định, tiền do `compute_compensation` tính từ order row và host duy nhất ghi ledger. Trên cùng 14 claim × 5 seed, workflow đạt pass rate/pass^5 1.000/1.000 với cost/success thấp nhất (34.29 VND mô phỏng), so với single 39.41, orchestrator 47.72 và swarm 49.57. Swarm đạt 0.986 pass rate do một trial tin delay trong ticket. Kết quả public ủng hộ lựa chọn workflow; cần đánh giá lại nếu hidden hoặc dữ liệu sản xuất cho thấy độ bao phủ nghiệp vụ của workflow chưa đủ.

## Có đáng đổi lấy multi-agent không

Hiện chưa có bằng chứng multi-agent tăng pass rate so với workflow trên bộ public; chúng có cost/success cao hơn. Nhóm giữ workflow. Có thể đổi nếu một bộ hidden/production đại diện cho thấy multi-agent cải thiện độ chính xác hoặc xử lý được trường hợp mà workflow bỏ sót đủ lớn để bù chi phí và độ phức tạp tăng thêm.

## Trace và bằng chứng

Trace workflow ghi `claim_view` đã lọc, `lookup_order` theo allowlist, quote và quyết định; không ghi ticket, principal ID, parent secret, raw header, số điện thoại, địa chỉ, ghi chú nội bộ hay tên hàng. Trace được ghi dưới `src/traces/` theo contract v1 và được kiểm qua ma trận public. Không có kết quả hidden trong workspace nên kết luận leak/bypass chỉ giới hạn ở kiểm tra public.

## Giới hạn

MockLLM là mô phỏng cố định theo seed, không đại diện cho model sản xuất; latency là `unavailable`. Benchmark có 14 claim public nên không bảo đảm pass rate hidden. Không có `student_drop/` để xác nhận cổng hidden ≥ 0.80. Số liệu được sinh từ `src/reports/benchmark.json` và `src/reports/crash.json`, không sửa tay.
