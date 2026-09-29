# Báo cáo cá nhân — K4-L3A Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Khi dẫn evidence, dùng đường dẫn tương đối, ví dụ `evidence/07-trace-waterfall.png`.

## 1. Thông tin học viên

- **Họ và tên:** Le Thi Hoai Thuong
- **MSSV:** 2A202602898
- **Lớp:** K4-L3A
- **Repository URL:** `https://github.com/thuongle06122004/K4-L3-DAY13-LeThiHoaiThuong-2A202602898-Monitoring-LLMOps`
- **Commit SHA cuối:** Cập nhật bằng SHA của commit chứa toàn bộ source, report và evidence ngay trước khi nộp LMS.
- **Challenge ID:** `day13-k4-l3a-monitoring-llmops-v1`
- **Tên project Langfuse cá nhân:** `day13-k4-l3a-2A202602898`

## 2. Evidence index

Điền đúng đường dẫn tới evidence thực tế. Có thể đổi tên hoặc dùng nhiều ảnh nếu cần.

| Evidence | Đường dẫn |
|---|---|
| Pytest cuối | `evidence/01-pytest.txt` |
| Log validator | `evidence/02-log-validator.txt` |
| Dashboard validator | `evidence/03-dashboard-validator.txt` |
| Structured log | `evidence/04-structured-log.txt` |
| PII redaction | `evidence/05-pii-redaction.txt` |
| Trace list | `evidence/06-trace-list.png` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png` |
| Prompt rollback | `evidence/10-prompt-rollback.png` |
| Dashboard runtime | `evidence/11-dashboard-overview.png` |
| Incident metric | `evidence/12-incident-metric.txt` |
| Incident log | `evidence/13-incident-log.txt` |
| Incident trace | `evidence/14-incident-trace.png` |

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---|---|---|
| `validate_logs.py` | Baseline đã chạy trong CP0 | 100/100; 0 PII leak (`evidence/02-log-validator.txt`) | Schema, correlation, enrichment và redaction đều pass. |
| `validate_dashboard.py` | Baseline đã chạy trong CP0 | 6/6 panels (`evidence/03-dashboard-validator.txt`) | Contract dashboard hợp lệ. |
| `pytest` | Baseline đã chạy trong CP0 | 24 passed (`evidence/01-pytest.txt`) | Regression tests pass. |
| Số traces hợp lệ | Chưa ghi trong report baseline | Cần xác nhận ≥10 trong Langfuse Cloud cá nhân | Không dùng trace của project khác. |
| Số PII leak | Baseline chưa đạt trước CP1 | 0 theo log validator | Runtime example: `evidence/05-pii-redaction.txt`. |
| Latency P95 / TTFT P95 | Chưa lưu số baseline | Challenge requests: 2,900–3,895 ms; TTFT 50–51 ms | Phân tích theo đúng cửa sổ incident tại mục 7. |
| Retrieval success rate | Chưa lưu số baseline | 5/5 trong challenge | `tool_success=true` trong `evidence/13-incident-log.txt`. |

## 4. Logging và PII

- **Cách tạo/nhận và truyền correlation ID:** Middleware nhận `x-request-id`; nếu thiếu sẽ sinh `req-<8-hex>`, bind vào structlog context, lưu ở `request.state`, và trả lại qua response header `x-request-id`.
- **Các metadata được ghi vào structured log:** `event`, `ts`, `level`, `correlation_id`, `user_id_hash`, `session_id`, `feature`, `model`, `env`; response bổ sung latency, TTFT, tokens, cost, quality, retrieval tool và trạng thái tool.
- **Cách bảo đảm PII được scrub trước khi ghi:** `scrub_event` xử lý đệ quy mọi string trước `JsonlFileProcessor`/JSON renderer. Rules che email, số điện thoại Việt Nam, CCCD và thẻ thanh toán; preview cũng đi qua `summarize_text`.
- **Cách kiểm chứng kết quả:** `evidence/04-structured-log.txt`, `evidence/05-pii-redaction.txt` và `evidence/02-log-validator.txt`. Runtime request PII có correlation ID `req-9aa898ed`; raw PII không xuất hiện trong log.

## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:** Cần mở project `day13-k4-l3a-2A202602898`, lọc workload hiện tại và lưu danh sách ≥10 trace tại `evidence/06-trace-list.png`.
- **Cấu trúc root/retrieval/generation observations:** `lab-agent-run` (agent) → `retrieve-context` (retriever) và `generate-answer` (generation). Generation ghi model, input/output preview đã scrub, input/output token và total cost.
- **Cách nối trace với log:** `correlation_id` được ghi vào root trace metadata và structured log. Ví dụ incident dùng `req-0129bb07`.
- **Prompt name:** `day13-chat`.
- **Version/label baseline:** Cần tạo v1 với labels `baseline`, `production` trong Langfuse Cloud và chụp `evidence/09-prompt-versions.png`.
- **Version/label candidate:** Cần tạo v2 với label `candidate`, chạy cùng input và lưu trace ID.
- **Trace ID của mỗi version:** Cần điền các trace ID thật sau khi chạy labels `baseline` và `candidate`; không ghi ID suy đoán.
- **Cách promote và rollback `production`:** Chuyển `production` từ v1 sang v2, chạy request, sau đó chuyển lại v1 và chụp `evidence/10-prompt-rollback.png`.

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:** Contract `config/dashboard.yaml` đạt 6/6: latency/TTFT, traffic, errors/retrieval success, cost, tokens và quality. Cần bổ sung ảnh runtime có time range, đơn vị và threshold tại `evidence/11-dashboard-overview.png`.
- **SLO và lý do chọn:** `config/slo.yaml` đặt SLO 99.5% request thành công và ≤3,000 ms trong 28 ngày, bảo vệ trải nghiệm chat tương tác nhưng vẫn cho phép retrieval/generation bình thường.
- **Cách tính error budget:** 100% − 99.5% = 0.5%; ví dụ 10,000 requests cho phép tối đa 50 request chậm hoặc không thành công.
- **Ba alert và runbook tương ứng:** `high_chat_error_rate`, `slow_chat_p95`, `retrieval_success_degraded` trong `config/alert_rules.yaml`; investigation/mitigation tại `docs/alerts.md`.

## 7. Điều tra challenge

- **Challenge ID:** `day13-k4-l3a-monitoring-llmops-v1` (cohort `K4`), incident `rag_slow`, feature `monitoring`, ngưỡng latency `2,000 ms`.
- **Khoảng thời gian điều tra:** `2026-09-29T07:56:33.047113Z` đến `2026-09-29T07:56:48.653152Z`.
- **Triệu chứng từ metrics:** Sau khi bật incident và chạy 5 query challenge với concurrency 5, cả 5 response đều vượt ngưỡng: latency `2,900–3,895 ms` (5/5 breach); không có error, TTFT `50–51 ms`, retrieval success `5/5`. Snapshot process-wide sau workload ghi nhận traffic `10`, latency P95 `1,631 ms`, nên điều tra dùng các `response_sent` trong đúng cửa sổ challenge thay vì P95 bị pha bởi traffic baseline.
- **Log line và correlation ID liên quan:** `data/logs.jsonl` lines 57–66. Correlation IDs: `req-0129bb07`, `req-2c46b770`, `req-76cd8bef`, `req-341ac72d`, `req-94452400`. Ví dụ `req-0129bb07` có `latency_ms=3895`, `tool_name=retrieval`, `tool_success=true`.
- **Trace ID và span gây ảnh hưởng:** Dùng các correlation IDs trên để lọc trace Langfuse; span cần kiểm tra là root `lab-agent-run` → retriever `retrieve-context`. Trace ID/screenshot Cloud cần được bổ sung tại `evidence/14-incident-trace.png` sau khi mở project cá nhân.
- **Root cause:** Incident `rag_slow` chủ động thêm `time.sleep(2.5)` trong hàm retrieval (`app/mock_rag.py`) trước khi trả tài liệu. Vì vậy retrieval vẫn thành công nhưng tổng latency tăng vượt SLO/challenge threshold.
- **Fix action:** Tắt incident bằng `python scripts/inject_incident.py --disable`; xác nhận health endpoint báo `rag_slow: false` và chạy request hồi phục.
- **Preventive measure:** Alert P95 latency khi vượt 3,000 ms trong 10 phút; điều tra theo correlation ID và waterfall để cô lập retriever chậm; áp dụng timeout/fallback cho vector store và rollback thay đổi retriever/index gây latency.

## 8. Giải thích và tự đánh giá

- **Một quyết định kỹ thuật quan trọng và lý do:** Tắt `capture_input/output` tự động của Langfuse và chỉ gửi preview qua `summarize_text`, để metadata hữu ích cho điều tra nhưng không gửi PII thô.
- **Một lỗi/blocker đã gặp:** Langfuse SDK v4 trong môi trường không cung cấp direct `start_as_current_span/generation` như tài liệu SDK cũ; Cloud evidence/prompt UI cũng cần quyền project cá nhân.
- **Cách tìm nguyên nhân và xử lý:** Dùng `@observe` cho child observation, sau đó dùng `update_current_span`/`update_current_generation`. Với CP3, lọc metric/log theo cửa sổ challenge và correlation ID đã cho thấy retriever chậm nhưng không lỗi.
- **Cách hiểu luồng Metrics → Logs → Traces:** Metrics chỉ ra latency tăng trong khoảng incident; logs thu hẹp về request và `correlation_id`; trace cùng ID cho phép xem waterfall để xác nhận retrieval/generation span nào gây ảnh hưởng; sau đó mới kết luận root cause và rollback/fallback.
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:** Prompt version/label cho phép release và rollback an toàn; token/cost hỗ trợ phát hiện cost spike; SLO/error budget chuyển tác động người dùng thành ngưỡng vận hành và alert; rollback giảm thời gian khôi phục khi prompt/model thay đổi gây regression.
- **Điều quan trọng nhất đã học:** Một incident chỉ có kết luận đáng tin khi metric, log và trace cùng dẫn đến cùng nguyên nhân, với PII được scrub ở mọi lớp telemetry.
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:** Cần bổ sung ảnh thật từ Langfuse Cloud (trace list/waterfall/metadata, prompt v1-v2/rollback, incident trace) và dashboard runtime. Không thay thế các evidence bắt buộc này bằng ảnh hoặc trace ID giả.

## 9. Checklist trước khi nộp

- [ ] Kết quả và evidence thuộc commit SHA cuối (cần commit/push các thay đổi hiện tại).
- [x] Evidence local dùng đường dẫn tương đối; cần bổ sung các ảnh Cloud/runtime còn thiếu.
- [ ] Incident evidence nối đúng metric → log → trace (metric/log đã có; cần ảnh trace Langfuse cùng correlation ID).
- [ ] Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret (cần chụp UI Cloud).
- [x] Repository chạy lại được theo README (`24 passed`, validators pass).
- [x] `.env` và `config/challenge.json` bị git-ignore; log validator báo 0 PII leak.
- [ ] URL repo và commit SHA cuối đã được nộp trên LMS/Codelabs.
