# Báo cáo cá nhân — K4-L3A Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Khi dẫn evidence, dùng đường dẫn tương đối, ví dụ `evidence/07-trace-waterfall.png`.

## 1. Thông tin học viên

- **Họ và tên:**
- **MSSV:**
- **Lớp:** K4-L3A
- **Repository URL:**
- **Commit SHA cuối:**
- **Challenge ID:** `day13-k4-l3a-monitoring-llmops-v1`
- **Tên project Langfuse cá nhân:** `day13-k4-l3a-<MSSV>`

## 2. Evidence index

Điền đúng đường dẫn tới evidence thực tế. Có thể đổi tên hoặc dùng nhiều ảnh nếu cần.

| Evidence | Đường dẫn |
|---|---|
| Pytest cuối | `evidence/01-pytest.png` |
| Log validator | `evidence/02-log-validator.png` |
| Dashboard validator | `evidence/03-dashboard-validator.png` |
| Structured log | `evidence/04-structured-log.png` |
| PII redaction | `evidence/05-pii-redaction.png` |
| Trace list | `evidence/06-trace-list.png` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png` |
| Prompt rollback | `evidence/10-prompt-rollback.png` |
| Dashboard runtime | `evidence/11-dashboard-overview.png` |
| Incident metric | `evidence/12-incident-metric.png` |
| Incident log | `evidence/13-incident-log.png` |
| Incident trace | `evidence/14-incident-trace.png` |

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---|---|---|
| `validate_logs.py` | | | |
| `validate_dashboard.py` | | | |
| `pytest` | | | |
| Số traces hợp lệ | | | |
| Số PII leak | | | |
| Latency P95 / TTFT P95 | | | |
| Retrieval success rate | | | |

## 4. Logging và PII

- **Cách tạo/nhận và truyền correlation ID:**
- **Các metadata được ghi vào structured log:**
- **Cách bảo đảm PII được scrub trước khi ghi:**
- **Cách kiểm chứng kết quả:**

## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:**
- **Cấu trúc root/retrieval/generation observations:**
- **Cách nối trace với log:**
- **Prompt name:**
- **Version/label baseline:**
- **Version/label candidate:**
- **Trace ID của mỗi version:**
- **Cách promote và rollback `production`:**

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:**
- **SLO và lý do chọn:**
- **Cách tính error budget:**
- **Ba alert và runbook tương ứng:**

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

- **Một quyết định kỹ thuật quan trọng và lý do:**
- **Một lỗi/blocker đã gặp:**
- **Cách tìm nguyên nhân và xử lý:**
- **Cách hiểu luồng Metrics → Logs → Traces:**
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:**
- **Điều quan trọng nhất đã học:**
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:**

## 9. Checklist trước khi nộp

- [ ] Kết quả và evidence thuộc commit SHA cuối.
- [ ] Tất cả ảnh/output mở được bằng đường dẫn tương đối.
- [ ] Incident evidence nối đúng metric → log → trace.
- [ ] Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret.
- [ ] Repository chạy lại được theo README.
- [ ] Không có secret, API key, PII thô hoặc evidence của người khác/lớp khác.
- [ ] URL repo và commit SHA cuối đã được nộp trên LMS/Codelabs.
