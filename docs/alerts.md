# Alert runbooks

Các alert đều dựa trên triệu chứng người dùng hoặc SLO. Gửi thông báo đến kênh Slack
được khai báo trong `config/alert_rules.yaml`; không đưa PII, prompt thô hoặc secret vào alert.

## High chat error rate

- **Severity / duration:** Critical khi error rate vượt 2% trong 5 phút.
- **SLI/SLO:** Tỷ lệ request hoàn thành thành công trong SLO `fast_successful_requests`.
- **Ảnh hưởng:** Người dùng nhận lỗi thay vì câu trả lời.
- **Kiểm tra đầu tiên:** (1) xác nhận error-rate panel và khoảng thời gian; (2) lọc `request_failed` trong `data/logs.jsonl`, nhóm theo `error_type`; (3) lấy một `correlation_id` rồi mở trace và kiểm tra retrieval/generation span.
- **Mitigation:** Roll back deployment hoặc prompt label thay đổi gần nhất; nếu lỗi retrieval tăng, chuyển sang fallback an toàn và thông báo search-platform on-call.
- **Owner:** `llmops-oncall`.

## Slow chat P95

- **Severity / duration:** Warning khi P95 latency vượt 3,000 ms trong 10 phút.
- **SLI/SLO:** Latency của `response_sent` trong SLO `fast_successful_requests`.
- **Ảnh hưởng:** Người dùng chờ phản hồi lâu, dù request vẫn có thể thành công.
- **Kiểm tra đầu tiên:** (1) xác nhận P95/P99 và TTFT trên latency panel; (2) lọc các `response_sent` chậm để lấy `correlation_id`; (3) so sánh thời lượng retrieval và generation trong trace waterfall.
- **Mitigation:** Giảm concurrency/traffic nếu cần, rollback prompt/model mới, hoặc tạm dùng retrieval fallback sau khi xác minh span chậm.
- **Owner:** `llmops-oncall`.

## Retrieval success degraded

- **Severity / duration:** Warning khi retrieval success rate thấp hơn 90% trong 10 phút.
- **SLI/SLO:** Guardrail `retrieval_success_rate_pct_min`.
- **Ảnh hưởng:** Câu trả lời có thể thiếu ngữ cảnh hoặc request thất bại.
- **Kiểm tra đầu tiên:** (1) xác nhận retrieval success trên errors panel; (2) lọc log theo `tool_name=retrieval` và `tool_success=false`; (3) mở trace theo `correlation_id` để đọc status/message của retrieval span.
- **Mitigation:** Chuyển sang corpus/cache fallback, kiểm tra trạng thái vector store và rollback thay đổi index/retriever gần nhất.
- **Owner:** `search-platform-oncall`.
