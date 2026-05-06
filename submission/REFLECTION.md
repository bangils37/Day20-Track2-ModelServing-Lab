# Reflection — Lab 20 (Personal Report)

> **Đây là báo cáo cá nhân.** Mỗi học viên chạy lab trên laptop của mình, với spec của mình. Số liệu của bạn không so sánh được với bạn cùng lớp — chỉ so sánh **before vs after trên chính máy bạn**. Grade rubric tính theo độ rõ ràng của setup + tuning của bạn, không phải tốc độ tuyệt đối.

---

**Họ Tên:** Nguyễn Bằng Anh
**Cohort:** 2A202600136
**Ngày submit:** 2026-05-06

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

- **OS:** Windows 11 (AMD64)
- **CPU:** Intel(R) Core(TM) i7-8665U CPU @ 1.90GHz
- **Cores:** 4 physical · 8 logical cores
- **CPU extensions:** SSE3, SSSE3, AVX, AVX2, F16C, FMA
- **RAM:** 15.7 GB
- **Accelerator:** CPU only
- **llama.cpp backend đã chọn:** CPU (AVX2 tuning)
- **Recommended model tier:** Qwen2.5-1.5B-Instruct (Q4_K_M)

**Setup story** (≤ 80 chữ): Setup trên Windows 11 khá mượt sau khi cài đặt Build Tools cho C++. Do máy không có GPU rời nên đã sử dụng backend CPU. Đã cài đặt llama-cpp-python[server] để hỗ trợ OpenAI-compat API. Gặp một chút vấn đề với Unicode trên terminal Windows nhưng đã fix bằng cách set PYTHONIOENCODING=utf-8.

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| qwen2.5-1.5b-instruct-q4_k_m.gguf | 1459 | 290 / 353 | 65.7 / 76.9 | 4450 / 5196 / 5347 | 15.2 |
| qwen2.5-1.5b-instruct-q2_k.gguf | 677 | 430 / 522 | 68.4 / 71.7 | 4774 / 4989 / 5040 | 14.6 |

**Một quan sát** (≤ 50 chữ): Thật ngạc nhiên khi Q2_K lại chậm hơn Q4_K_M một chút ở cả TTFT và TPOT trên máy này. Có lẽ do overhead của việc dequantize Q2 phức tạp hơn trên CPU i7-8665U đã triệt tiêu lợi ích từ việc giảm memory bandwidth. Nên dùng Q4_K_M vì chất lượng tốt hơn hẳn.

---

## 3. Track 02 — llama-server load test

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.12 | 28000 | 45000 | 45000 | 0 |
| 50 | 0.13 | 38000 | 48000 | 48000 | 0 |

**KV-cache observation** (từ `record-metrics.py`): peak `llamacpp:kv_cache_usage_ratio` ở concurrency 50 = 0.18, nghĩa là với model 1.5B và context 2048, server vẫn còn khá nhiều RAM trống cho KV cache dù response time đã rất cao do nghẽn CPU.

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** stub: localhost only
- **N17 (Data pipeline):** stub: in-memory dict
- **N18 (Lakehouse):** stub: SQLite (via toy_docs metadata)
- **N19 (Vector + Feature Store):** stub: TOY_DOCS (keyword overlap retrieval)

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: 0 ms (using toy keyword overlap)
- retrieve: 0.1 ms
- llama-server: 3912.0 ms

**Reflection** (≤ 60 chữ): Bottleneck chính nằm ở phần inference của llama-server (chiếm 99.9% thời gian). Việc retrieval diễn ra cực nhanh trên in-memory docs. Điều này khớp với kỳ vọng vì inference trên CPU cho model 1.5B với prompt dài tốn nhiều compute nhất.

---

## 5. Bonus — The single change that mattered most

**Change:** Tối ưu hóa số lượng thread (`-t`) cho phù hợp với physical core (4).

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before (logical=8): 12.5 tok/s
after (physical=4): 15.2 tok/s
speedup: ~1.2×
```

**Tại sao nó work** (1–2 đoạn ngắn): Việc sử dụng hyperthreading (8 logical cores) thực tế làm chậm quá trình inference do tranh chấp tài nguyên bộ đệm (cache) và băng thông bộ nhớ (memory bandwidth) giữa các luồng. Khi giới hạn về 4 physical cores, mỗi lõi có tài nguyên cache riêng biệt hơn và giảm bớt overhead quản lý luồng, giúp throughput tăng khoảng 20%.

---

## 6. (Optional) Điều ngạc nhiên nhất

Sự chênh lệch hiệu năng giữa Q4 và Q2 không như lý thuyết (Q2 nhanh hơn). Thực tế trên phần cứng cũ, thuật toán dequantize có thể là nút thắt cổ chai thay vì băng thông RAM.

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [ ] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [ ] Ít nhất 6 screenshots trong `submission/screenshots/`
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [x] Repo trên GitHub ở chế độ **public**
- [x] Đã paste public repo URL vào VinUni LMS
