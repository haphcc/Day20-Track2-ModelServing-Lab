# 02 Server Results

This file captures the committed Track 02 load-test summary for the current CPU-only run.

## Load test summary

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.16 | 23000 | 48000 | 48000 | 0 |
| 50 | 0.19 | 28000 | 45000 | 45000 | 0 |

## Notes

- The installed `llama_cpp.server` wheel in this workspace does not expose a Prometheus `/metrics` endpoint, so `record-metrics.py` could not collect `llamacpp:kv_cache_usage_ratio` samples.
- The load-test numbers above come from the committed locust runs in `benchmarks/locust-10.log` and `benchmarks/locust-50.log`.