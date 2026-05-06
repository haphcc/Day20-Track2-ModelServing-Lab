# Reflection - Lab 20 (Personal Report)

> This is a personal report. Each student runs the lab on their own laptop. Your numbers are not comparable to classmates - the grade depends on how clearly you can explain setup and tuning on your own machine.

---

**Name:** _HAMESEMAN_
**Cohort:** _A20-K1_
**Submit date:** _2026-05-06_

---

## 1. Hardware spec (from `00-setup/detect-hardware.py`)

> Paste the output of `python 00-setup/detect-hardware.py` here, or fill it manually:

- **OS:** _Windows 11_
- **CPU:** _Intel i5-1135G7_
- **Cores:** _4 physical / 8 logical_
- **CPU extensions:** _AVX2_
- **RAM:** _15.7 GB_
- **Accelerator:** _CPU only_
- **llama.cpp backend selected:** _CPU_
- **Recommended model tier:** _Qwen2.5-1.5B-Instruct (Q4_K_M)_

**Setup story** (<= 80 words): what needed to change to make the lab work on this machine.

Windows 11 + PowerShell, using `.venv` and the prebuilt `llama-cpp-python` wheel to avoid a local C++ build. This machine is CPU-only for the lab, so I stayed on the Qwen2.5 1.5B Q4_K_M path and used the 4 physical cores as the main tuning knob.

---

## 2. Track 01 - Quickstart numbers (from `benchmarks/01-quickstart-results.md`)

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| qwen2.5-1.5b-instruct-q4_k_m.gguf | 1681 | 134 / 178 | 51.2 / 52.8 | 3343 / 3493 / 3526 | 19.5 |
| qwen2.5-1.5b-instruct-q2_k.gguf | 684 | 251 / 290 | 43.0 / 46.4 | 2908 / 3210 / 3240 | 23.3 |

**One observation** (<= 50 words): Q2_K decodes a bit faster, but Q4_K_M gives better output on long prompts. On this machine the throughput gap is not large enough to justify the lower quality, so Q4_K_M is the better default.

---

## 3. Track 02 - llama-server load test

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.16 | 23000 | 48000 | 48000 | 0 |
| 50 | 0.19 | 28000 | 45000 | 45000 | 0 |

**KV-cache observation** (from `record-metrics.py`): I could not get a peak ratio from the current `llama_cpp.server` wheel because this build does not mount `/metrics`, so `record-metrics.py` collected no samples. If I rerun with a metrics-enabled native `llama-server`, I expect the ratio to rise at concurrency 50 and especially on the long-rag requests.

---

## 4. Track 03 - Milestone integration

- **N16 (Cloud/IaC):** stub: localhost-only single-node stack; no real cluster
- **N17 (Data pipeline):** stub: in-memory query loop in `pipeline.py`; no real batch job
- **N18 (Lakehouse):** stub: no Delta/Iceberg table; data lives in `TOY_DOCS`
- **N19 (Vector + Feature Store):** stub/partial: keyword-overlap retrieval on `TOY_DOCS`, not a real vector index

**Where the time goes** (measured with `time.perf_counter` in `pipeline.py`):

- embed: 0.0 ms (no real embed step)
- retrieve: 0.0-0.1 ms
- llama-server: 3576.8-14504.9 ms

**Reflection** (<= 60 words): where is the bottleneck? Does it match expectations?

The bottleneck is almost entirely in llama-server generation, not retrieval. That matches the toy setup: retrieval is effectively free, while token generation dominates end-to-end latency. If I optimize further, I would work on prompt length, quantization, and batching before touching the retrieval stub.

---

## 5. Bonus - The single change that mattered most

> Most important section. Pick one change from the bonus track that gave the biggest speedup on your machine.

**Change:** _Keep the stack CPU-only and use Q4_K_M instead of Q2_K for the main model_

**Before vs after** (paste 2-3 lines from sweep output):

```
before: qwen2.5-1.5b-instruct-q2_k.gguf - 23.3 tok/s
after:  qwen2.5-1.5b-instruct-q4_k_m.gguf - 19.5 tok/s
speedup: ~0.84x
```

**Why it works** (1-2 short paragraphs):

This machine is CPU-only, so the real bottleneck is memory bandwidth and decode latency, not GPU compute. Q2_K pushes tok/s higher, but Q4_K_M keeps answer quality better while remaining fast enough on 4 physical cores. For this lab, I would rather keep the runtime stable and the outputs trustworthy than squeeze out a few more tok/s.

---

## 6. (Optional) Most surprising thing

The most surprising thing was how close to free the retrieve step was, compared with the time spent in generation. I also learned that telemetry depends heavily on the exact launcher/build: the Python `llama_cpp.server` wheel in this workspace does not expose `/metrics`.

---

## 7. Self-graded checklist

- [x] `hardware.json` committed
- [x] `models/active.json` committed (or path snapshot pasted in section 1)
- [x] `benchmarks/01-quickstart-results.md` committed
- [x] `benchmarks/02-server-results.md` committed
- [ ] `benchmarks/bonus-*.md` committed (at least 1 sweep)
- [ ] At least 6 screenshots in `submission/screenshots/`
- [ ] `make verify` exit 0 (run right before push)
- [ ] Repo on GitHub is public
- [ ] Public repo URL pasted into VinUni LMS

---

Important: the repo must be public until grades are released. If it is private, the grader cannot see it.