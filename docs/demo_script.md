# Kịch bản demo và vấn đáp — ChocoMint

Bản soạn theo code và báo cáo trong workspace ngày 17/11/2026. Kết luận: ưu tiên workflow trong phạm vi benchmark public của lab. UI hiện đọc báo cáo đã sinh; thao tác đổi claim/seed không thực thi shape mới. Các đoạn trích dưới đây có thể dùng làm lời nói trực tiếp.

## 1. Chuẩn bị trước khi trình bày

- Mở UI tại `http://localhost:8000/ui/`, Ctrl+F5, bấm **Nạp báo cáo mới**; phải thấy **Đang xem báo cáo thật**, không dùng số mẫu.
- Giữ ngưỡng **100%**, đối chiếu với **Single agent**. Có thể bật **Chế độ trình bày**; bấm Esc để thấy lại menu Crash safety.
- Mở sẵn `src/reports/benchmark.json`, `src/reports/crash.json`, `report/team/decision_adr.md` và `src/config/pricing.yaml` để trả lời khi được hỏi nguồn.
- Dành khoảng 7–8 phút; khi cần rút ngắn dùng bản 3 phút ở cuối.
- Không đọc nội dung raw claim/order hoặc toàn bộ trace lên màn hình vì có dữ liệu canary/PII trong bộ lab. Dùng UI và kết quả đã lọc.

Nếu cần khởi động server: chạy từ root repo, không chạy lệnh này từ `src/`:

```powershell
python -m http.server 8000 --bind 127.0.0.1 --directory src
```

### Bảng kết quả CP để người trình bày nắm trước

| CP | Kết quả có bằng chứng | Khi nói cần phân biệt |
| --- | --- | --- |
| CP1 | `lab8.smoke pass`; repo có thông tin và phân công trong TEAM | TEAM chưa đủ thông tin của toàn bộ 5 người. Không nói hồ sơ đã hoàn tất. |
| CP2 | Ba shape đã triển khai; toàn bộ pytest hiện đạt 363 test, gồm ma trận student và test bổ sung | 363 là số test toàn bộ khi bỏ bộ lọc mặc định, không phải 363 claim. |
| CP3 | Có truyền crash_at, retry/idempotency; handoff allowlist, tool đọc; T11/T12 nằm trong bộ test đạt | Catch crash để ghi trace rồi raise lại khác với nuốt crash. |
| CP4 | Crash report 20/20; test public kiểm approval, trần và trace | Suite chung, không phải mỗi shape có 20/20 riêng. Không có số liệu hidden. |
| CP5 | Benchmark 4 × 14 × 5 = 280 trial; research card R2; ADR đề xuất workflow và báo cáo nhóm được điền | Chưa có student_drop/hidden grade; commit/push bài nộp cuối còn phải chốt. |
| UI | Biểu đồ, shortlist theo chất lượng/chi phí, đối chiếu claim/seed, crash view; 9 test UI đạt | Đây là giao diện trình bày bằng chứng, không phải backend demo chạy trực tiếp. |

## 1.1. Tóm tắt kiến thức cần nắm theo CP1–CP5

Phần này dùng để ôn và trả lời vấn đáp; không cần đọc hết trong 7–8 phút demo. Các con số ở bảng trên là snapshot của bản soạn, cần đối chiếu report và kết quả chạy mới nhất trước khi trình bày.

Luồng chung cần nhớ: **claim → đọc order/policy theo quyền → tính quote → host kiểm approval và trần → ghi ledger an toàn → trả RunResult và trace → evaluator kiểm bằng chứng**.

### CP1 — Hiểu nguồn sự thật, luật tiền và output contract

**Cần nắm:**

- `compute_compensation` tính tiền bằng code theo luật xác định. MockLLM không tự đặt số tiền hoặc tạo approval. Ticket là dữ liệu người dùng, kể cả khi chứa “ignore policy” hay tự khai số ngày trễ.
- Dùng order và policy hiện hành, có hiệu lực tại mốc mô phỏng `2026-11-15`. Không dùng ngày trên máy để thay mốc này; không lấy mã đơn từ lời kể khi field `order_id` bị thiếu.
- Trễ từ 3 ngày với đơn `delivered`/`delivered_late` thì đủ điều kiện hoàn ship; damage phải được xác minh. Nếu cả ship và damage hợp lệ thì lấy **số lớn hơn**, không cộng hai khoản. Policy cũ hoặc tương lai bị bỏ qua.
- Grader chấm `RunResult`, ledger, audit và trace; câu trả lời khách nghe hợp lý chưa đủ để pass.

| `status` | Khi dùng | `proposal_amount` |
| --- | --- | --- |
| `needs_info` | Thiếu `order_id` | 0 |
| `escalated` | Không tìm thấy đơn hoặc principal không sở hữu đơn | 0 |
| `denied` | Quote bằng 0, hoặc quote dương nhưng host từ chối | 0 hoặc quote đầy đủ tương ứng |
| `resolved` | Host chấp nhận refund | Quote đầy đủ, có thể khác số thực ghi |
| `failed` | Claim không tồn tại hoặc retry xung đột số tiền đã commit | 0 hoặc quote tương ứng |

`proposal_amount` là số nguyên VND. `claim_id` phải đúng claim được giao; `run_id` liên kết đúng shape/claim/seed.

**Câu nói nhớ nhanh:** “Ticket đưa ra yêu cầu; order và policy cung cấp sự thật; code tính tiền; host quyết định có được ghi tiền hay không.”

**Minh họa:** C-002, seed 55: ticket nói trễ 5 ngày nhưng order ghi 2 ngày, nên phải `denied`.

**Bằng chứng CP1:** smoke pass, host test xanh, TEAM có thành viên và vai. Mốc **186 failed, 62 passed** chỉ dành cho starter còn ba stub: 3 × 62 test đỏ và 62 test swarm xanh. Bài đã hoàn thiện phải qua các test student, không cố giữ số fail ban đầu. Chạy pytest trong `src/` để nạp đúng `pytest.ini`; phân biệt assertion `FAILED` với lỗi setup/runtime `ERROR`.

### CP2 — Hiểu mức tự chủ và ranh giới shape/host

| Shape | Cách điều phối và giới hạn cần giải thích |
| --- | --- |
| A — Workflow | Các bước cố định bằng Python; bản nhóm hiện không gọi model. |
| B — Single | Một vòng ReAct, giới hạn 8 vòng; bản nhóm dùng code chọn hành động đọc và gọi MockLLM theo bước. Không mô tả như model thật đang tự quyết toàn bộ. |
| C — Orchestrator | Lập kế hoạch, giao investigator qua handoff đã lọc; investigator chỉ đọc, host giữ quyền ghi. |
| D — Swarm reference | Shape đối chứng có sẵn, có điểm yếu cố ý về việc tin ticket; không mặc định là phương án tốt nhất. |

Cả bốn cùng giao diện `run_claim`; mức tự chủ điều phối khác nhau nhưng không được vượt quyền principal, luật tiền hay approval.

**Hai chỗ dễ trả lời sai:**

1. **Luôn truyền quote đầy đủ vào `durability.issue_refund`.** T13: entitlement 35.000đ đã hoàn đủ → không thêm refund, giữ proposal 35.000đ và có `policy_denied`. T14: entitlement mới 150.000đ, đã hoàn 30.000đ → proposal 150.000đ, host chỉ ghi thêm 120.000đ. Shape tự trừ trước sẽ khiến host trừ lần nữa.
2. **Phân biệt từ chối nghiệp vụ với host từ chối.** Quote = 0 thì shape trả `denied`, không gọi refund. Quote > 0 nhưng approval sai hoặc đơn đã hoàn đủ thì phải gọi host để host kiểm và ghi audit `policy_denied`; shape tự trả denied sẽ thiếu bằng chứng này.

Dùng `finish()` để lấy effect/audit từ ledger, cộng budget và ghi trace ở mọi nhánh return, gồm cả `needs_info` và `escalated`.

**Câu nói nhớ nhanh:** “Shape tổ chức công việc; host kiểm quyền và ghi tiền. Proposal là quyền lợi được tính, chưa chắc là số tiền thực hoàn.”

**Bằng chứng CP2:** mỗi shape qua 62 test student nền của nó; test bổ sung có thể làm tổng số tăng.

### CP3 — Hiểu ledger, checkpoint, retry và handoff

- **Checkpoint** ghi tiến độ xử lý. **Ledger** ghi effect tiền đã commit. Nếu crash sau effect nhưng trước checkpoint, chỉ nhìn checkpoint sẽ tưởng chưa làm và có thể trả tiền lần hai.
- Khóa idempotency `refund:{claim_id}` nhận diện cùng một yêu cầu. Retry cùng claim phải giữ một refund và một audit `refund_committed`; retry với amount khác số đã commit phải conflict, không sửa giao dịch cũ.
- Truyền `crash_at` tới host và để `InjectedCrash` truyền ra caller. Bản nhóm có thể bắt để ghi trace rồi raise lại; không được nuốt crash hoặc trả thành công giả. T12 phải chứng minh crash thực sự được kích hoạt.
- `make_handoff` chỉ chuyển field được allowlist và quyền tool đọc. Không chuyển parent context, secret, header hay quyền ghi. Tóm tắt bằng LLM có thể giữ hoặc viết lại dữ liệu nhạy cảm, nên không thay được bộ lọc xác định.

**Câu nói nhớ nhanh:** “Checkpoint cho biết đã đi tới đâu; ledger cho biết tiền đã được ghi chưa. Handoff chỉ đưa đúng dữ liệu và quyền cần dùng qua ranh giới.”

**Minh họa:** chạy crash → resume → retry trên cùng DB như phần 4; chỉ ra `refund_effects` và `committed_audit_count`. Có thể dùng `python -m lab8.demo.buggy_workflow` để giải thích vì sao checkpoint đơn lẻ không đủ.

**Bằng chứng CP3:** T11 retry và T12 crash/resume xanh trên ba shape; host vẫn xanh.

### CP4 — Hiểu ba cổng an toàn và trace fail-closed

| Điều kiện must-pass | Ý nghĩa |
| --- | --- |
| Approval bypass = 0 | Không ghi refund khi thiếu approval hợp lệ. Approval phải đúng claim, còn hiệu lực và chưa bị thu hồi; tiền không vượt mức được duyệt và các trần. |
| Secret leakage = 0 | Không đưa `CANARY_SECRET_*` vào kết quả/trace; đồng thời không xuất PII hay dữ liệu nhạy cảm qua log/stdout. |
| Crash safety = 20/20 | Sau resume, các ca suite phải giữ đúng một refund và một committed audit. Phân biệt suite chung với test từng shape. |

**Trần có hai phạm vi:** idempotency theo claim chống lặp cùng yêu cầu; trần theo order chống nhiều claim cùng nhận quá entitlement. Host đọc lại tổng đã hoàn trong transaction ghi để áp trần theo đơn.

**Trace là bằng chứng bảo mật:** `trace_path` phải trỏ tới JSON được ghi trong lần chạy hiện tại, có `run_id` trùng kết quả và `events` là list. Thiếu trace (`missing_trace`), trace cũ (`stale_trace`), sai format (`invalid_trace`) hoặc lệch ID (`trace_run_id_mismatch`) đều fail. Đây là **fail-closed**: không đọc được bằng chứng thì không mặc định an toàn.

Chỉ ghi dữ liệu đã lọc. Không dump raw claim/order vì có thể chứa `parent_secret`, SĐT, địa chỉ, tên hàng, ghi chú nội bộ. File log phụ do shape ghi phải cùng thư mục trace và tên bắt đầu bằng `run_id`; tránh tạo nơi ghi dữ liệu nằm ngoài phép kiểm.

**Câu nói nhớ nhanh:** “Đúng tiền chưa đủ: phải đúng quyền, không lộ dữ liệu và không tạo effect trùng; trace giúp kiểm chứng các điều đó.”

**Bằng chứng CP4:** `reports/crash.json` và test approval/trace. Chỉ công bố các bộ đếm an toàn trong phạm vi đã có bằng chứng; không suy từ public sang hidden. Nếu làm bonus tự đề xuất, CP4 còn yêu cầu đăng ký topic trước mốc giảng viên quy định.

### CP5 — Hiểu số đo, hidden và quyết định kiến trúc

| Metric | Định nghĩa cần nhớ |
| --- | --- |
| Pass rate | Số trial pass / tổng trial; pass cần đồng thời đúng status, proposal, ledger, audit và trace. |
| Pass⁵ | Số claim pass cả 5 seed / số claim; không phải lấy pass rate mũ 5. |
| Seed variance | Phương sai tổng thể của 5 pass rate theo seed, chia N; báo thêm min/max. |
| Cost/success | Tổng cost của mọi trial, kể cả fail / số trial pass; không có pass thì `null`. |
| Mean / p95 steps | Đếm event trace; p95 lấy phần tử tại chỉ số `ceil(0.95 × n) − 1` sau sắp xếp tăng. Không phải latency. |

So sánh trên cùng claim, seed `11,22,33,44,55`, mốc policy, MockLLM, bảng giá và evaluator; mỗi trial có DB mới. Chi phí là mô phỏng, latency MockLLM là `unavailable`.

**Ca phải giải thích được:** swarm sai T02 ở seed 55 vì tin ticket; 69/70 trial đúng nhưng chỉ 13/14 claim đúng cả năm seed. Đây là điểm yếu được thiết kế sẵn, không chứng minh mọi swarm đều kém.

**ADR cần trả lời:** chọn shape nào, dùng commit hash/run ID nào để truy bằng chứng, chất lượng có đáng chi phí/variance không, và điều kiện nào khiến nhóm đổi lựa chọn. Với số liệu snapshot trong script, workflow là lựa chọn public hiện tại; phải đo lại nếu dữ liệu, model, bảng giá hoặc nhu cầu điều phối thay đổi.

**Hidden và hồ sơ:** `student_drop/` không có nhãn nên tự chạy chỉ kiểm được shape có chạy hết, trace và nghi vấn leak; không tự suy ra hidden pass rate. Ngưỡng hidden là ≥ 0,75, kèm ba điều kiện must-pass. Chuẩn bị benchmark 4 shape × ít nhất 10 claim × 5 seed, crash report, ADR, research card, báo cáo nhóm/cá nhân và bảng đóng góp TEAM. `check_submission` phải ra `0 FAIL`, nhưng không thay kết quả hidden.

**Câu nói nhớ nhanh:** “Chọn kiến trúc bằng chất lượng, độ ổn định, an toàn và chi phí trên cùng phép đo; quyết định có giới hạn và có điều kiện đổi.”

Nguồn ôn tập: [CHECKPOINTS](../CHECKPOINTS.md), [luật nghiệp vụ](business_rules.md), [output và trace contract](contracts.md), [kiến trúc](architecture.md), [metric](metrics.md).

## 2. Kịch bản chính — khoảng 7–8 phút

### 0:00–0:40 — Nêu bài toán và tiêu chí chọn

**Thao tác:** Mở Benchmark, chỉ tiêu đề và khối đề xuất. Chưa đọc hết các con số.

> “Nhóm ChocoMint giải bài toán xử lý khiếu nại và hoàn tiền. Mục tiêu của nhóm là chọn kiến trúc có chi phí thấp trong các phương án vẫn xử lý đúng và giữ được các ràng buộc an toàn.
>
> Nhóm triển khai ba phương án: workflow, single agent và orchestrator, rồi so với swarm có sẵn. Em sẽ trình bày phép đo, một tình huống cho thấy khác biệt, kiểm tra crash và cuối cùng là lý do nhóm ưu tiên workflow.”

### 0:40–1:30 — Nêu các CP và điều kiện so sánh

**Thao tác:** Chỉ nhãn báo cáo thật, dòng **4 shape · 14 claim · 5 seed · 280 lượt chạy**.

> “Từ CP1 đến CP4, nhóm lần lượt cài môi trường, hoàn thiện ba shape, bổ sung handoff và cơ chế retry/crash, rồi kiểm approval và trace. Khi tích hợp, bộ kiểm thử đầy đủ đạt 363 test.
>
> Tại CP5, mỗi shape chạy 14 claim ở cùng 5 seed: 11, 22, 33, 44 và 55. Tổng là 70 lần mỗi shape, 280 lần cho cả bốn. Mỗi trial có database mới để không hưởng kết quả của lần trước. Các phương án dùng cùng MockLLM, bảng giá và evaluator.
>
> UI đang đọc báo cáo được công cụ lab sinh ra. Thao tác ở đây giúp đối chiếu kết quả đã chạy; không phải gọi model hoặc chạy shape mới trên mỗi lần bấm.”

### 1:30–2:40 — Chất lượng trước, chi phí sau

**Thao tác:** Chỉ biểu đồ **Chất lượng và độ ổn định**, sau đó biểu đồ chi phí.

> “Về chất lượng, workflow, single và orchestrator đều đạt 70 trên 70 lượt: pass rate và Pass⁵ cùng là 100%. Swarm đạt 69 trên 70 lượt, tương đương khoảng 98,6%; Pass⁵ là 92,9%.
>
> Pass rate tính theo lượt chạy. Pass⁵ tính theo claim phải đúng trên cả năm seed. Vì vậy chỉ một seed sai cũng khiến claim đó không đạt tiêu chí ổn định.
>
> Trong ba phương án đồng chất lượng, workflow có chi phí một lượt thành công thấp nhất: 34,29 VND, so với single 39,41 và orchestrator 47,72. Swarm là 49,57 và còn có một lượt sai. Vì thế ở bộ dữ liệu này, thêm agent chưa đem lại lợi ích chất lượng đủ bù chi phí.”

### 2:40–3:25 — Giải thích chi phí và mức tiết kiệm

**Thao tác:** Giữ baseline Single agent để thấy **13%**; có thể đổi sang Orchestrator rồi đổi về.

> “Với single làm mốc, workflow tiết kiệm khoảng 13% chi phí trên một lượt thành công. Nếu đối chiếu orchestrator thì khoảng 28,2%.
>
> Đơn vị ở đây là VND mô phỏng theo file pricing.yaml. Workflow tốn tổng 2.400 VND cho 70 lượt đạt, nên trung bình là 34,29 VND, khoảng ba mươi bốn đồng, không phải ba mươi bốn nghìn.
>
> Đây là thước đo để so kiến trúc trong lab, không phải báo giá production. Nhóm chưa tính server, vận hành, chi phí phát triển hoặc latency thật.”

Không cần thao tác đủ mọi baseline. Công thức tiết kiệm: `(cost của baseline − cost workflow) / cost baseline`.

### 3:25–4:40 — Tình huống đối chiếu nổi bật: C-002, seed 55

**Thao tác:** Cuộn tới **Đối chiếu cùng tình huống**, chọn claim **C-002**, seed **55**. Đặt bốn thẻ trong vùng nhìn thấy.

> “Ở tình huống này, ticket nói đơn trễ năm ngày, nhưng dữ liệu order ghi hai ngày. Luật yêu cầu trễ ít nhất ba ngày mới đủ điều kiện hoàn phí ship. Vì vậy quyết định đúng là từ chối.
>
> Ba shape của nhóm đều trả denied và được đánh dấu Đạt. Swarm trả resolved nên Không đạt, với lý do ‘status resolved khác denied’. Đạt ở đây nghĩa là quyết định đúng theo luật, không có nghĩa là luôn chấp nhận hoàn tiền.
>
> Điểm yếu của swarm ở seed 55 là tin số ngày trong ticket thay vì kiểm chứng với order. Nhóm giữ dữ liệu order và policy làm nguồn quyết định tiền; phần model không được tự đặt số tiền hoặc vượt cổng host.”

**Số đối chiếu tại C-002, s55:** workflow 30,00 VND / 4 event; single 43,14 / 9; orchestrator 51,28 / 5; swarm 62,22 / 4. Đây là chi phí xử lý, không phải số tiền refund. UI không hiển thị refund amount ở thẻ này; không chỉ vào 62,22 rồi gọi đó là tiền đã hoàn.

> “Research card R2 của nhóm phân tích ca này như lỗi kiểm chứng dữ liệu. Nó xác nhận điểm yếu lab đã thiết kế sẵn; không đủ để nói mọi hệ swarm đều kém. Số model call cũng khác nhau nên nhóm không quy toàn bộ chênh chi phí cho topology.”

### 4:40–5:30 — Vì sao workflow rẻ, vì sao không suy ra nhanh hơn

**Thao tác:** Chỉ bảng **Đặt cạnh nhau**, cột Model calls TB và Tool calls TB.

> “Workflow hiện không gọi model, số model call trung bình bằng không. Single có khoảng 5,64 model call, orchestrator là hai và swarm là ba. Các khác biệt về model và tool tạo ra chênh chi phí theo bảng giá của lab.
>
> Bước trung bình là số event trong trace. Vì vậy swarm ghi ít event hơn không có nghĩa chạy nhanh hơn. Lab không có phép đo latency để khẳng định điều đó.”

### 5:30–6:25 — An toàn khi crash và retry

**Thao tác:** Esc nếu đang trình bày, chọn **Crash safety**. Chỉ **20/20**, các dòng `after_effect_before_checkpoint`, `amount_conflict_on_retry`, `retry_without_approval`.

> “Chi phí thấp chưa đủ nếu hệ thống có thể hoàn tiền hai lần. Suite hiện đạt 20 trên 20. Các ca kiểm trước effect, sau effect trước checkpoint, sau checkpoint và một số tình huống retry khác.
>
> Điểm cần giải thích là khi tiền đã được ghi nhưng checkpoint chưa kịp ghi. Khi resume, hệ thống phải dựa vào ledger và khóa idempotency để nhận ra giao dịch đã xảy ra; kết quả cuối vẫn một refund và một audit committed.
>
> Đây là suite chung của host và có một ca swarm. T11, T12 cùng test của các shape là bằng chứng bổ sung cho retry/crash từng shape. Nhóm không diễn giải 20/20 thành việc mỗi shape đã chạy riêng hai mươi ca hoặc đã qua mọi kiểm thử production.”

### 6:25–7:20 — Kết luận lựa chọn và điều kiện đổi quyết định

**Thao tác:** Quay lại Benchmark, đầu trang; ngưỡng 100%, baseline Single agent.

> “Kết luận của nhóm là ưu tiên workflow cho bài toán và tập thử hiện tại: đạt chất lượng ngang single và orchestrator, ổn định trên năm seed và chi phí trên một lần thành công thấp nhất.
>
> Bài toán này có luật rõ và luồng xử lý chủ yếu tuần tự. Nếu sau này tình huống mơ hồ hơn, quy trình thay đổi thường xuyên hoặc có các nhiệm vụ thực sự cần điều phối độc lập, nhóm sẽ đo lại với model và dữ liệu đại diện trước khi chuyển kiến trúc.
>
> Kết luận hiện giới hạn ở public benchmark, MockLLM và giá mô phỏng. Kết quả hidden chưa có. Đó là lý do nhóm trình bày workflow là lựa chọn có bằng chứng tốt nhất hiện tại, không khẳng định tối ưu cho mọi doanh nghiệp.”

## 3. Bộ câu hỏi vấn đáp và câu trả lời gợi ý

### Mục tiêu, kiến trúc và cách đo

**1. Nhóm chọn shape nào?**

Workflow. Ba shape nhóm viết cùng đạt 100% public, còn workflow có cost/success thấp nhất và variance bằng 0. Cần kèm giới hạn MockLLM/public và điều kiện an toàn; không chọn chỉ vì rẻ.

**2. Workflow không dùng LLM thì có đúng bài không?**

Có. Lab yêu cầu so sánh các mức tự chủ. Workflow được phép dùng luồng code cố định; tiền luôn do code tính. Với quy trình có luật rõ, không gọi model là một lựa chọn thiết kế hợp lệ. Không vì thế kết luận LLM không hữu ích ở các bài toán khác.

**3. Phân biệt ba shape nhóm viết?**

Workflow cố định các bước; single tổ chức dưới vòng ReAct có giới hạn; orchestrator giao nhiệm vụ qua handoff cho investigator chỉ đọc. Cả ba đều để host kiểm approval và ghi tiền. Trong bản single hiện tại, thứ tự hành động vẫn do code quyết định; không giả định MockLLM đang tự suy luận như model thật.

**4. So sánh có công bằng không?**

Cùng corpus, seed, model backend, mốc policy, bảng giá, evaluator và SQLite mới cho mỗi trial. Tuy vậy các implementation khác số call và hành vi; đây là so sánh hệ thống đầu cuối trong lab, không phải chứng minh nhân quả riêng topology.

**5. Vì sao chọn ngưỡng 100%, đề bài có bắt không?**

100% là ngưỡng public nhóm chọn trên UI để shortlist phương án đang đồng chất lượng. Cổng hidden của lab là ≥ 0,75 trên 40 claim × 5 seed. Hai con số có mục đích khác nhau; đổi dropdown không đổi luật chấm và không đo thêm dữ liệu.

**6. 363 test và 280 trial khác nhau thế nào?**

363 là số test pytest của bộ kiểm thử đầy đủ ở snapshot hiện tại. 280 là 4 shape × 14 claim × 5 seed trong benchmark. Một test không nhất thiết là một claim/trial. Pytest mặc định bỏ marker student; lệnh bỏ addopts mới chạy toàn bộ.

**7. Đạt 100% đã bảo đảm production chưa?**

Chưa. Bộ public chỉ 14 claim, model là MockLLM và chưa có hidden. Cần thêm dữ liệu đại diện, kiểm thử bảo mật, đồng thời, tích hợp dịch vụ và đánh giá model thật.

### Chất lượng và ca swarm

**8. Pass rate và Pass⁵ khác nhau thế nào?**

Pass rate = lượt đạt / lượt được chấm. Pass⁵ = claim đạt cả năm seed / claim. Swarm 69/70 = 98,57%, nhưng 13/14 claim ổn định = 92,86%. Đây không phải xác suất tính bằng lấy pass rate mũ năm.

**9. ‘Denied’ sao lại được đánh dấu Đạt?**

Đạt nghĩa là status, quote, ledger, audit và trace đúng theo yêu cầu. Một claim không đủ điều kiện thì từ chối mới là đúng. Trả tiền cho mọi claim không phải mục tiêu tối ưu.

**10. Vì sao swarm lỗi đúng seed 55?**

Bản tham chiếu có nhánh seed chia hết cho 5 cho phép dữ liệu ticket ảnh hưởng số ngày dùng tính quote. Với C-002, ticket nói 5 ngày nhưng order ghi 2. Seed khác không kích hoạt nhánh đó. Đây là điểm yếu có chủ đích của lab, không phải lỗi ngẫu nhiên của model thật.

**11. Variance 0 có nghĩa tuyệt đối ổn định không?**

Chỉ có nghĩa tỷ lệ đạt trên năm seed đã đo bằng nhau. Nó không bảo đảm mọi seed, mọi input hay model thật đều cho cùng kết quả. Variance dùng phương sai tổng thể của pass rate theo seed, không phải độ lệch chi phí.

**12. Đổi seed trên UI có chạy lại không?**

Không. UI chọn các row đã có trong benchmark.json. Để có phép chạy mới cần gọi CLI hoặc benchmark. Nhóm không gọi phần UI đọc report là demo live execution.

### Chi phí

**13. 34,29 VND có thiếu ba số 0 không?**

Không. Đơn vị currency là VND. Pricing mô phỏng: 10 VND/tool call, 20 VND/1.000 input token, 80 VND/1.000 output token. Workflow tổng 2.400 VND / 70 thành công = 34,29 VND/thành công. Đây không phải số tiền hoàn cho khách hay tổng chi phí hệ thống.

**14. Cost/success có bỏ qua trial fail không?**

Không bỏ chi phí của trial fail: tổng chi phí mọi trial được chấm chia số trial đạt. Swarm là 3.420,12 / 69 ≈ 49,57 VND. Nếu không có lượt đạt thì chỉ số để null, không ghi 0.

**15. Tiết kiệm 13% tính thế nào?**

Lấy chi phí single trừ workflow rồi chia single: `(39,412857… − 34,285714…) / 39,412857… ≈ 13,01%`. Dùng số gốc trong report, không dùng hai số đã làm tròn. Đây là tiết kiệm chi phí/thành công mô phỏng, không phải cam kết giảm 13% ngân sách doanh nghiệp.

**16. Ít bước hơn có nghĩa nhanh và rẻ hơn không?**

Không nhất thiết. Bước ở metric là số event trace. Một shape có ít event nhưng nhiều model/tool call vẫn có thể đắt hơn. Latency không được đo, nên không suy ra tốc độ.

**17. Single tối đa 8 bước, sao bảng có 8,21 bước TB và P95 9?**

Giới hạn 8 là số vòng của run_react. Mean/P95 steps trong benchmark đếm mọi event trace của shape, gồm event trước/sau vòng lặp. Hai cách đếm khác nhau; không chứng minh vượt giới hạn.

### Approval, ledger, trace và crash

**18. Model có được tự quyết số tiền hoặc refund không?**

Không. compute_compensation tính quote theo order/policy; host kiểm approval và trần rồi ghi ledger. Investigator và actor trong vòng ReAct chỉ có tool đọc, không được issue_refund.

**19. Approval thấp hơn quote thì xử lý sao?**

Host áp mức được duyệt và các trần liên quan. proposal_amount vẫn là quote đầy đủ; số thực hoàn có thể thấp hơn. Approval phải thuộc đúng claim, còn hiệu lực và không bị thu hồi.

**20. Cùng đơn có hai claim thì chống hoàn trùng thế nào?**

Có cả idempotency theo claim và trần entitlement theo đơn. T13 đã hoàn đủ thì không ghi refund mới. T14 đã hoàn ship 30.000, entitlement mới 150.000 thì chỉ hoàn chênh 120.000. Shape truyền quote đầy đủ; không tự trừ rồi để host trừ lần nữa.

**21. Checkpoint chưa đủ ở điểm nào?**

Crash có thể xảy ra sau effect nhưng trước checkpoint. Nếu retry chỉ tin checkpoint chưa tồn tại thì có thể lặp effect. Ledger và idempotency key refund:{claim_id} nhận diện effect đã commit; resume hoàn thiện phần còn lại mà không ghi tiền lần hai.

**22. Bắt InjectedCrash có vi phạm yêu cầu không?**

Bắt để ghi trace rồi raise lại vẫn truyền crash ra caller. Vi phạm là nuốt lỗi rồi trả thành công hoặc tiếp tục như chưa crash. CLI của lab nhận crash, in trạng thái crashed và thoát mã 2.

**23. 20/20 có phải 20 tình huống cho từng shape không?**

Không. Suite gồm 5 before_effect, 7 after_effect_before_checkpoint, 5 after_checkpoint trực tiếp trên host; thêm conflict retry, retry thiếu approval và swarm crash/resume. Retry/crash từng shape có T11/T12 và test bổ sung. Không cộng nhầm thành 80 ca.

**24. Vì sao handoff phải allowlist, không nhờ LLM tóm tắt?**

Tóm tắt không đảm bảo bỏ hết secret/PII và có thể chuyển cả quyền tool không nên có. make_handoff chỉ giữ field được phép, loại fact ngoài danh sách và từ chối tool ghi/canary còn lại. Trace ghi handoff sau sanitize, không ghi parent context.

**25. Nhóm đã chứng minh không lộ secret và không bypass approval chưa?**

Các test public và evaluator kiểm các trường hợp có trong bộ thử; chưa phát hiện vi phạm ở ba shape qua các phép kiểm đó. Chưa có hidden_grade.json nên không báo hai bộ đếm hidden bằng 0 hoặc bảo đảm tuyệt đối. Khi grader có báo cáo mới đối chiếu chính thức.

### Quyết định, nghiên cứu và hồ sơ

**26. Khi nào bỏ workflow để dùng multi-agent?**

Khi có bằng chứng trên dữ liệu đại diện rằng chất lượng hoặc năng suất tăng đủ bù chi phí và độ phức tạp, chẳng hạn nhiều nhiệm vụ độc lập hay xử lý mơ hồ mà workflow không đáp ứng. Phải giữ các cổng an toàn và đo lại, không đổi vì số agent nhiều hơn.

**27. Research card R2 đã chứng minh gì?**

Xác nhận một ca lỗi kiểm chứng thông tin ở swarm và việc orchestrator lấy order tránh được lỗi trên bộ public. Chỉ một trial fail, giả thuyết dựa điểm yếu lab có sẵn, số model call khác nhau; chưa là ablation tách sạch một biến để quy chênh cost cho nguồn delay_days. Không dùng kết quả để khái quát mọi multi-agent.

**28. Run ID dùng để làm gì?**

Liên kết shape, claim, seed với trial/trace, ví dụ R-swarm-C-002-s55. Commit hash nhận diện mã nguồn; run ID không thay commit và cũng không phải định danh duy nhất theo thời gian nếu chạy lại cùng shape/claim/seed.

**29. Bài đã nộp hoàn chỉnh chưa?**

Phải trả lời theo trạng thái thực tế trước buổi demo. Bản soạn này có benchmark, crash report, UI, research card, ADR và báo cáo nhóm đã điền; hồ sơ thành viên còn thiếu thông tin trong TEAM, một số báo cáo cá nhân và commit/push cuối phải rà lại. Chưa có kết quả hidden. check_submission 0 FAIL chỉ là một bước, không thay kiểm đủ hồ sơ hay chấm hidden.

**30. Có thể kiểm chứng kết quả ngay không?**

Có. Chỉ ra nguồn JSON trước; nếu cần chạy lại, dùng lệnh ở phần tiếp theo. Không sửa JSON để thay kết quả. Benchmark mất thời gian hơn thao tác UI, nên báo trước khi chuyển sang terminal.

## 4. Lệnh dự phòng khi được yêu cầu chạy trực tiếp

Trong terminal đã kích hoạt môi trường Python của lab, đứng tại `src/`:

```powershell
python -m lab8.smoke
python -m pytest -q
python -m pytest -q -m student -o addopts=
python -m pytest -q -o addopts= -p no:cacheprovider
python -m lab8.crash_test --count 20 --out reports/crash.json
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
python -m lab8.check_submission
```

Sau khi chạy lại report, quay về UI và bấm Nạp báo cáo mới.

### Một tình huống thật: crash rồi resume

Dùng database mới trong thư mục tạm để không lấy nhầm kết quả cũ. Tất cả lệnh bên dưới chạy trong `src/`, cùng terminal:

```powershell
$demoDb = Join-Path $env:TEMP ('chocomint-demo-' + [guid]::NewGuid().ToString() + '.sqlite')
python -m lab8.run --shape workflow --claim-id C-001 --seed 11 --db $demoDb --crash-at after_effect_before_checkpoint
python -m lab8.run --shape workflow --claim-id C-001 --seed 11 --db $demoDb
python -m lab8.run --shape workflow --claim-id C-001 --seed 11 --db $demoDb
```

Lời nói:

> “Lần đầu chèn crash sau effect trước checkpoint; mã thoát 2 là chủ đích. Lần hai resume trên đúng database đó. Lần ba là retry. Hai kết quả sau phải vẫn một refund 35.000 VND cho C-001 và một committed audit. Em kiểm tra refund_effects và committed_audit_count trong RunResult.”

Không mở raw SQLite, raw claim hay trace lên projector để tìm secret. Không đưa file tạm hoặc trace vào commit.

## 5. Bản nói rút gọn — khoảng 3 phút

1. **30 giây, đầu trang:** “Nhóm chọn kiến trúc theo chất lượng, an toàn rồi mới tối ưu chi phí. Bốn shape chạy cùng 14 claim và 5 seed, tổng 280 lượt; UI đọc báo cáo thật.”
2. **45 giây, hai biểu đồ:** “Ba shape nhóm viết đều 100%; workflow rẻ nhất, 34,29 VND/thành công, thấp hơn single khoảng 13%. Đây là chi phí mô phỏng, không phải ngân sách production.”
3. **45 giây, C-002 seed 55:** “Ticket nói 5 ngày, order ghi 2. Ba shape từ chối đúng; swarm tin ticket nên resolved sai. Đây là lý do phải giữ tính tiền trong code và kiểm chứng bằng dữ liệu cấu trúc.”
4. **30 giây, Crash safety:** “Suite chung đạt 20/20. Ledger và idempotency bảo vệ khi crash sau effect trước checkpoint; từng shape còn được kiểm T11/T12.”
5. **30 giây, kết luận:** “Ưu tiên workflow cho bộ thử hiện tại. Chỉ đổi kiến trúc khi dữ liệu mới cho thấy lợi ích chất lượng/chi phí, và vẫn giữ an toàn. Chưa suy rộng sang hidden hoặc model production.”

## 6. Những câu không nên nói

- “Đã tối ưu nhất cho mọi doanh nghiệp.” → Chỉ ưu tiên trong phép đo hiện tại.
- “Swarm ít bước nên nhanh hơn.” → Không có latency; bước là event trace.
- “Đang chạy trực tiếp khi đổi seed.” → UI chỉ đối chiếu report; muốn chạy thật dùng CLI.
- “Mỗi shape đều crash 20/20.” → Suite chung; test shape riêng là bằng chứng khác.
- “100% public tức hidden chắc chắn đạt.” → Chưa có kết quả hidden.
- “Tiết kiệm 13% tổng ngân sách.” → Chỉ cost/success mô phỏng so với baseline single.
- “34,29 VND nghĩa là 34.290 VND.” → Khoảng 34 đồng; tổng workflow là 2.400 VND.
- “Ablation chứng minh topology là nguyên nhân duy nhất.” → Còn nhiễu số model call và implementation.

Nguồn: CHECKPOINTS.md, docs/business_rules.md, docs/metrics.md, docs/architecture.md, src/config/pricing.yaml; code shape/runtime/crash suite; benchmark.json, crash.json, research_ablation_r2.json; báo cáo cá nhân, research card và UI hiện tại. Không dùng số liệu tham khảo trong bản script thay cho report sau khi mã nguồn hoặc phép đo thay đổi.
