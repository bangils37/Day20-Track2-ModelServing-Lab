# 01 - Quickstart Results

Settings: n_threads=4, n_ctx=2048, n_batch=512, n_gpu_layers=0.

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|---:|---:|---:|---:|---:|
| qwen2.5-1.5b-instruct-q4_k_m.gguf | 1459 | 290 / 353 | 65.7 / 76.9 | 4450 / 5196 / 5347 | 15.2 |
| qwen2.5-1.5b-instruct-q2_k.gguf | 677 | 430 / 522 | 68.4 / 71.7 | 4774 / 4989 / 5040 | 14.6 |

## Observations

- Tren Intel i7-8665U, ban Q4_K_M cho hieu nang on dinh o muc 15.2 tok/s.
- That thu vi khi Q2_K co TTFT cao hon Q4_K_M (430ms vs 290ms), co the do CPU overhead khi xu ly nen Q2 lon hon.
- Decode rate cua Q4_K_M van rat tot (15.2 tok/s), hoan toan su dung duoc cho chat real-time.
- n_threads=4 (physical cores) giup toi uu bang thong RAM tot hon so voi dung logical cores.