# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

## Bảng phân tích chi tiết

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, Run ID: `E-seeded_regression-20261117-30da` |
| Triệu chứng | Grader Lab 8 `trial_pass` báo FAIL (`status resolved != denied`). Ledger thực tế bị trừ `refund_vnd=20000`, `refund_count=1` trong khi nhãn kỳ vọng là `denied`, `refund_vnd=0`. Reply gửi khách: `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."` |
| Bước sai đầu tiên | **Event 3** · kind: `model_override` · payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Các bước trước hoàn toàn chính xác: Event 1 (`handoff`) khởi tạo đúng scope; Event 2 (`investigator`) gọi `lookup_order` thành công và nhận dữ kiện chuẩn từ DB: `delay_days: 2`, `status: delivered` (trễ 2 ngày, không đủ điều kiện bồi thường theo điều khoản tối thiểu 3 ngày). Tại Event 3, agent tin lời tự khai trong ticket ("giao trễ 5 ngày") và đè dữ liệu DB bằng `model_override`. |
| Failure mode | **FM-03** (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Trên shape `fixed` (cùng claim C-002@11), event `model_override` không xảy ra. Agent giữ nguyên `delay_days: 2`, Event 3 trả về quote bồi thường 0đ (`reason: no_entitlement`), Event 4 trả lời từ chối hoàn tiền hợp lệ, ledger không bị ghi nhận hoàn tiền. Grader PASS (ok). |
| Regression test | `DbFactKept` trong `src/lab9/graders.py` và test `test_override_event_breaks_db_fact` trong `src/tests/test_lab9_graders.py`. Grader trả về `value=False` trên `seeded_regression` và `value=True` trên `fixed`. |
| Chưa chắc chắn | Trace chỉ chứng minh agent đã ghi nhận sự kiện `model_override` từ kết quả parse ticket; trace không chứng minh được liệu trong một model LLM sinh văn bản tự do, model có nhận thức được sự mâu thuẫn giữa ticket và DB hay chỉ đơn thuần ưu tiên văn cảnh hội thoại gần nhất. |

## Quy tắc áp dụng
- **Ledger trước reply**: Đọc dữ liệu biến động số dư trong ledger trước khi đọc nội dung câu chữ phản hồi; phản hồi văn phong lịch sự vẫn có thể che giấu khoản hoàn sai.
- **Xác định bước sai nguyên nhân gốc**: Chỉ định chính xác số thứ tự event trên trace, trích xuất payload minh chứng thay vì kết luận cảm tính.
- **Có kiểm thử hồi quy**: Mọi trace case phải đi kèm assert tự động hóa được tích hợp trong bộ test/eval harness.
