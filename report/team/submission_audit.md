# Rà soát bài nộp — 17/11/2026

Đối chiếu SUBMISSION.md sau PR #12–#16, trên nhánh main. Phân biệt việc kiểm tra repo với việc mỗi cá nhân tự nộp LMS; không tự đánh dấu hoàn thành bước chưa có bằng chứng.

| Yêu cầu | Bằng chứng / trạng thái |
| --- | --- |
| Tên repo và quyền truy cập | Repo public K5-L3B-DAY08-ChocoMint-FromWorkflowsToAgents; Sang xác nhận lớp chính thức L3B. Mẫu SUBMISSION ghi L3A, giữ lớp thật theo xác nhận. |
| TEAM và đóng góp | Năm thành viên, đủ vai/commit/test. Test Linh đã tích hợp vào main tại f526841, nguồn 268d137, giữ tác giả. |
| Báo cáo cá nhân | Đủ năm file theo MSSV; cập nhật Sang, Linh và Quỳnh Anh theo bằng chứng tích hợp. Phần tự đánh giá/AI từng người cần tự xác nhận. |
| Báo cáo nhóm / ADR / research card | Đã điền; ADR chọn workflow, tham chiếu snapshot 6d9b8c2 và run ID thật; R2 có report ablation. |
| Ba shape và test thêm | Có trong nhánh nộp. Toàn bộ pytest 419 passed, 6 xfailed; 9 test UI pass. Xfail strict là lỗi grader cố ý B7. |
| Benchmark | Chạy lại bằng lab8.benchmark, 4 shape × 14 claim × 5 seed; không sửa tay JSON. |
| Crash suite | Chạy lại bằng lab8.crash_test, 20/20. |
| Giới hạn trang | Xuất thử bản cuối bằng Chrome headless A4, Segoe UI 10pt, lề 14mm: báo cáo nhóm 3/5 trang, bonus 2/2 trang. Thay kiểu in có thể đổi số trang. |
| File cấm / credential | Không có trace, SQLite, student_drop, .venv, .env hoặc instructor pack trong danh sách Git theo dõi. Không thấy mẫu AWS access key, OpenAI project key hoặc private key trong lần quét file hiện tại; không phải bảo đảm toàn bộ lịch sử không có secret. |
| Hidden self-check tùy chọn | PR #14 đưa reports/hidden_selfcheck.json vào main: 600 trial không nhãn, toàn bộ reason unlabeled; không lỗi thực thi/trace/cảnh báo canary. Không có điểm hidden chính thức. |
| Bonus | Có code, hai bộ test/JSON và một bonus_report.md gộp cả permutation/isolation, kết luận và giới hạn. Gate hidden ≥ 0,80 và điểm bonus vẫn chờ giảng viên. |
| Checker bonus | PR #16 mở rộng tên B1–B7, không đổi metric/grader/host. Checker local nhận B7; checker gốc phía giảng viên có thể vẫn cần cập nhật tương thích. |
| LMS | Sang xác nhận đã nộp link. Chưa có xác nhận của Nhung, Quỳnh Anh, Thảo Quyên và Linh; từng người phải tự nộp cùng link trước deadline. |

## Lệnh kiểm tra

Trong src/, sau khi kích hoạt venv:

```powershell
python -m pytest -q -p no:cacheprovider
python -m pytest -q -o addopts= -p no:cacheprovider
python -m pytest -q tests/test_bonus_grader_audit.py -p no:cacheprovider
python -m lab8.check_submission
```

Từ root: `node --test src/ui/report.test.mjs src/ui/integration.test.mjs`.

Snapshot mã nguồn đã commit trước lần cập nhật hồ sơ: 6d9b8c248509ed1815269e3ff7e9a5c720f6426f. PR #16 được giải conflict tại 117f920, đã merge và push lên main. Hồ sơ nằm ở commit sau, tránh yêu cầu file tự chứa hash của chính commit đang tạo.

## Ngoài phạm vi xác nhận tự động

Không có quyền đọc trạng thái LMS từng người hay điểm hidden giảng viên. Không suy deadline từ ngày trong file mẫu khi chưa biết lịch học/gia hạn thực tế. Lần quét Git chỉ kiểm trạng thái nhánh nộp và các mẫu credential phổ biến, không phải bảo đảm không tồn tại secret trong toàn bộ lịch sử.


Bản tích hợp sau PR #16 giữ cả test permutation và isolation; hai script sinh hai JSON riêng. Toàn bộ 419 passed, 6 xfailed. Đã xuất thử báo cáo nhóm bằng Chrome headless, A4, Segoe UI 10pt, lề 14mm: 3 trang; số trang có thể thay đổi khi đổi kiểu in.
