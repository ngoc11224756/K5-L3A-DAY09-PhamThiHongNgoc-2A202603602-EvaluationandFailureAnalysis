# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Observed FAIL | Rogan-Gladen FAIL | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | 0.280 | 0.300 | Đạt (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | **dùng** (làm metric theo dõi/cổng phụ) |
| helpful | 100 | 0.375 | 1.000 | 0.300 | 0.090 | 0.300 | **Không đạt** (κ=0.375 < 0.6, R=0.300 < 0.7) | **loại** (không dùng làm gate) |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | `H-008` ("Em đã hoàn tiền cho anh rồi nha, anh check giúp em.") | PASS | FAIL | Người | Văn phong suồng sã ("anh/em", "nha", "check"). Sửa rubric `tone_ok`: cấm đại từ thân mật và từ lóng; bắt buộc xưng "LogiX" / "anh/chị". |
| 2 | `H-029` ("Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm.") | PASS | FAIL | Người | Dù có "Dạ" mở đầu nhưng câu "không cần nhắn thêm" mang tính xua đuổi. Sửa rubric: kiểm tra thái độ toàn câu, không chỉ dựa vào từ lịch sự đầu câu. |
| 3 | `H-042` ("Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ.") | FAIL | PASS | Người | Câu trả lời lịch sự, rõ ràng và có thêm SLA. Sửa rubric: không bắt buộc lặp tên thương hiệu nếu đã có chủ ngữ/thông tin rõ. |
| 4 | `H-051` ("Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ.") | FAIL | PASS | Người | Judge phạt nhầm từ "refund". Sửa rubric: cho phép thuật ngữ "refund" nếu ngữ cảnh lịch sự. |
| 5 | `H-070` ("Dạ, LogiX đã hoàn 90.000đ cho đơn O-909 ạ 😂😂") | PASS | FAIL | Người | Dùng emoji cười trong phản hồi tài chính là thiếu chuyên nghiệp. Sửa rubric: cấm emoji không phù hợp. |
| 6 | `H-094` ("Da, LogiX da hoan 40.000đ cho don O-341, xin loi vi su bat tien.") | PASS | FAIL | Người | Thiếu dấu tiếng Việt. Sửa rubric: yêu cầu viết tiếng Việt có dấu chuẩn. |
| 7 | `HL-016` ("...thành thật xin lỗi... rất trân trọng... Một lần nữa LogiX xin lỗi...") | PASS | FAIL | Người | Judge `helpful` bị verbosity bias (thấy xin lỗi dài dòng thì chấm PASS), bỏ sót lỗi lặp thừa. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` không được dùng làm gate vì trên `judge_helpful_public.csv` (`n=100`, `tp=9, fp=0, fn=21, tn=70`), judge chỉ đạt **Recall = 0.300** và **κ = 0.375** (dưới sàn R ≥ 0.7, κ ≥ 0.6). Với `fn=21`, judge bỏ sót 70% ca lỗi thực tế — nếu dùng làm gate sẽ báo `observed_fail_rate = 0.090` trong khi tỉ lệ lỗi thực sau hiệu chỉnh Rogan-Gladen là `0.300` (gấp >3 lần), tạo cảm giác an toàn giả.
- **Tiêu chí dùng được:** `tone_ok` (`tp=24, fp=4, fn=6, tn=66`, κ=0.757, P=0.857, R=0.800) đạt đủ 3 sàn, nhưng vẫn cần cập nhật rubric cho 6 trường hợp bất đồng ở trên và hiệu chỉnh Rogan-Gladen (`0.280 → 0.300`).
- **Rủi ro còn lại:** judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới để có tập held-out thật.
