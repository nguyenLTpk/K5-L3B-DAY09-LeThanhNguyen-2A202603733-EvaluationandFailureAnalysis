# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `seeded_regression-C-002-11` |
| Triệu chứng | Grader `TrialPass` và `RefundAmountCorrect` fail (lab8: `status resolved != denied`). Ledger ghi nhận `refund_vnd=20000`, `refund_count=1` trong khi nhãn kỳ vọng `refund_vnd=0`, `status=denied`. Reply nói `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."` |
| Bước sai đầu tiên | Event 3: `model_override {"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) hợp lệ; Event 2 (`investigator`) truy vấn đúng dữ liệu DB từ đơn hàng O-202 (`delay_days: 2`). Đến Event 3, agent tự ý tin ticket ghi 5 ngày và override số ngày trễ lên 5, dẫn đến tính toán quote 20.000đ ở Event 4. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Trên shape `fixed` (C-002@11), không hề có event `model_override`. Event 3 trực tiếp là `quote {"amount_vnd": 0, "reason": "no_entitlement"}`, dẫn tới kết quả `denied` hoàn toàn chính xác. |
| Regression test | `test_override_event_breaks_db_fact` (kiểm tra grader `DbFactKept`) và test so sánh `RefundAmountCorrect` / `ApprovalBypass`; đã xác nhận fail trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận output event `model_override` chứ không chứa prompt/hidden chain-of-thought giải thích vì sao mô hình lại chọn tin ticket hơn DB. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
