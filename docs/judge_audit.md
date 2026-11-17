# Judge audit

Chạy kiểm định:
```bash
python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5
python -m lab9.judge --csv data/eval/judge_helpful_public.csv --audit 5
```

## Kết quả hiệu chuẩn (Calibration Results)

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | **ĐẠT** (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | **Dùng** |
| helpful | 100 | 0.375 | 1.000 | 0.300 | **KHÔNG ĐẠT** (κ=0.375 < 0.6, R=0.300 < 0.7) | **Loại** |

- Ma trận nhầm lẫn `judge_tone_public`: `TP=24, FP=4, FN=6, TN=66`. Tỉ lệ lỗi quan sát: 0.28, tỉ lệ lỗi hiệu chỉnh (Rogan-Gladen): 0.30.
- Ma trận nhầm lẫn `judge_helpful_public`: `TP=9, FP=0, FN=21, TN=70`. Tỉ lệ lỗi quan sát: 0.09, tỉ lệ lỗi hiệu chỉnh (Rogan-Gladen): 0.30.

## 5 case audit thủ công trên bất đồng

| # | Case ID | Judge nói | Người nói | Ai đúng? | Nội dung reply & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | **Người đúng** | `"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."`<br>**Phân tích:** Ngôn ngữ quá suồng sã, xưng hô "Em - anh", dùng từ "nha", "check giúp em" không đạt tiêu chuẩn chuyên nghiệp của LogiX.<br>**Sửa rubric:** Bổ sung quy tắc đại từ nhân xưng bắt buộc "LogiX - anh/chị/quý khách", cấm xưng hô "em - anh", cấm dùng từ đệm "nha". |
| 2 | H-017 | PASS | FAIL | **Người đúng** | `"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."`<br>**Phân tích:** Dùng từ "bạn ơi", "rồi nha" mang tính khẩu ngữ tùy tiện trong CSKH doanh nghiệp.<br>**Sửa rubric:** Cấm các cụm từ khẩu ngữ thân mật không trang trọng như "bạn ơi", "rồi nha". |
| 3 | H-029 | PASS | FAIL | **Người đúng** | `"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."`<br>**Phân tích:** Thái độ cộc lốc, có sắc thái xua đuổi và chặn kênh giao tiếp của khách hàng ("không cần nhắn thêm").<br>**Sửa rubric:** Thêm tiêu chí cấm phát ngôn gây ức chế, phủ định nhu cầu trao đổi của khách hàng. |
| 4 | H-042 | FAIL | PASS | **Người đúng** | `"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."`<br>**Phân tích:** Câu trả lời đầy đủ thông tin, lịch sự, cung cấp thời gian ước tính hợp lý cho khách hàng. Judge bắt lỗi quá máy móc.<br>**Sửa rubric:** Quy định rõ việc nêu thời hạn xử lý ngân hàng (3–5 ngày) là thông tin hữu ích và được phép, không bị coi là lỗi tone. |
| 5 | H-051 | FAIL | PASS | **Người đúng** | `"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."`<br>**Phân tích:** Sử dụng từ mượn "refund" khá phổ biến trong thương mại điện tử, ngữ điệu lịch sự, khách hàng vẫn hiểu rõ ý định.<br>**Sửa rubric:** Cho phép các thuật ngữ quen thuộc trong ngành như "refund", "ship" nếu câu từ vẫn giữ được sự tôn trọng và rõ nghĩa. |

## Kết luận & Quyết định

1. **Lý do loại `judge_helpful` làm CI gate:**
   - Recall chỉ đạt **0.300** (bỏ lọt tới 21/30 trường hợp fail do con người gán nhãn).
   - Hệ số Cohen's Kappa chỉ đạt **0.375** (dưới ngưỡng sàn 0.60), cho thấy độ tương đồng ngoài may rủi rất yếu.
   - Judge quan sát tỉ lệ lỗi là 9% trong khi thực tế con người đánh giá là 30%. Nếu dùng `helpful` làm gate, 70% các phản hồi vô dụng hoặc gây hại sẽ lọt qua cổng kiểm soát.

2. **Rủi ro còn lại đối với `judge_tone`:**
   - Dữ liệu đánh giá hiện tại chỉ là tập replay công khai (`judge_tone_public.csv`).
   - Cần tiếp tục thu thập nhãn của chuyên viên đánh giá độc lập trên tập dữ liệu người dùng thật (held-out set) định kỳ để tránh overfitting rubric và trôi dạt phân phối (distribution drift).
