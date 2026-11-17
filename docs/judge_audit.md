# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | Dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (κ=0.375 < 0.6, R=0.300 < 0.7) | Loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Đề xuất sửa Rubric |
|---|---|---|---|---|---|
| 1 | H-008 (`Em đã hoàn tiền cho anh rồi nha, anh check giúp em.`) | PASS | FAIL | Người đúng | Xưng hô suồng sã ("anh/em/nha/check"), không đúng chuẩn CSKH chuẩn mực của LogiX. Cần thêm rubric cấm từ lóng, tiếng bồi và quy định đại từ nhân xưng chuẩn. |
| 2 | H-017 (`Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha.`) | PASS | FAIL | Người đúng | "Bạn ơi... rồi nha" thiếu tính trang trọng, chuyên nghiệp. Cần bổ sung ví dụ negative few-shot cho judge về văn phong suồng sã. |
| 3 | H-029 (`Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm.`) | PASS | FAIL | Người đúng | Giọng điệu cộc cằn, ra lệnh ("không cần nhắn thêm") gây cảm giác khó chịu cho khách hàng. Judge bị lừa bởi từ "Dạ, vui lòng". Cần bổ sung tiêu chí cấm câu từ gắt gỏng. |
| 4 | H-042 (`Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ.`) | FAIL | PASS | Người đúng | Câu trả lời lịch sự, rõ ràng, có thông tin thời gian tiền về tài khoản. Judge bị over-sensitive (báo nhầm FP). Cần nới rubric công nhận thông tin đối soát ngân hàng là hợp lệ. |
| 5 | HL-016 (`Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này...`) | PASS | FAIL | Người đúng | Câu trả lời xin lỗi sáo rỗng nhưng không giải quyết vấn đề của khách hàng (thiếu helpful). Judge chấm PASS vì thấy văn phong lịch sự. Cần tách rõ tiêu chí "helpful" yêu cầu phải có action/thông tin cụ thể thay vì chỉ xin lỗi. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` bị loại hoàn toàn khỏi release gate.
  - *Lý do:* Ma trận nhầm lẫn `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}` cho thấy Recall chỉ đạt `0.300` (bỏ lọt 21/30 trường hợp FAIL của con người, tương đương 70% lỗi bị lọt qua). Hệ số Cohen's $\kappa = 0.375$ rất thấp, không đảm bảo độ tin cậy để làm chốt chặn tự động.
- **Rủi ro còn lại:** Cả hai judge hiện mới chỉ được đánh giá trên tập dữ liệu replay công khai ($n=100$). Cần thu thập thêm tập dữ liệu held-out thực tế từ người dùng mới để kiểm tra tính tổng quát hóa trước khi đưa vào sản xuất.
