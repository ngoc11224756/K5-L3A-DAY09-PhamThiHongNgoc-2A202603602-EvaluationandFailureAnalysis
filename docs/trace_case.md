# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

## Case đã phân tích: `seeded_regression C-002@11`

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID `E-seeded_regression-20261117-bed2` |
| Triệu chứng | `lab8 grader` (`pass_rate`), `db_fact_kept`, `refund_amount_correct` đều FAIL (`status resolved != denied`); ledger ghi `refund_vnd=20000`, `refund_count=1` trong khi label là `0` và `denied`; reply nói `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."`. |
| Bước sai đầu tiên | Event #3 `model_override`: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event #1 (`handoff`) chuyển đúng `REQ-C-002` với các tool chỉ đọc; Event #2 (`investigator`) truy vấn `lookup_order` trả về đúng dữ liệu DB cho `O-202` (`visible: true`, `status: "delivered"`). Chỉ đến Event #3 agent mới lấy `delay_days: 5` từ ticket người dùng để ghi đè sự thật DB, dẫn tới Event #4 (`quote`) tính ra `20000` và hoàn tiền sai. |
| Failure mode | `FM-03` (`ticket_trust` — tin ticket hơn DB) trong [failure_taxonomy.md](file:///Users/macbook/Desktop/c%C3%A1c%20b%C3%A0i%20lab/K5-L3A-PhamThiHongNgoc-2A202603602-Day9-Evaluation-and-Failure-Analysis/docs/failure_taxonomy.md) |
| Đối chứng | Trên `fixed C-002@11` (run ID `E-fixed-20261117-bed2`): sau Event #1 (`handoff`) và Event #2 (`investigator`), **không có** Event `model_override`; Event #3 (`quote`) trả `{"amount_vnd": 0, "reason": "no_entitlement"}`, Event #4 (`reply`) từ chối đúng chính sách, ledger `refund_vnd=0`. |
| Regression test | `test_c002_ticket_override_regression` trong [test_lab9_graders.py](file:///Users/macbook/Desktop/c%C3%A1c%20b%C3%A0i%20lab/K5-L3A-PhamThiHongNgoc-2A202603602-Day9-Evaluation-and-Failure-Analysis/src/tests/test_lab9_graders.py) dùng `DbFactKept` + `RefundAmountCorrect` (và gate chặn `seeded_regression` trên `pass_rate`, `refund_amount_correct`, `reply_matches_ledger`, còn `fixed` pass 100%). |
| Chưa chắc chắn | Trace trên MockLLM không chứng minh được hành vi prompt injection phức tạp hơn trên LLM thật (ngoài mẫu parse `delay_days` cố định). |
