# Results: 427 KV Cache Stress Test

**Main goal:** Analyze LLM KV-cache stress under long contexts  
**Tools:** Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py

Reference results from the validation run on the system shown in each section (recorded 2026-11-22). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 80GB HBM3 (81559 MiB) · Intel(R) Xeon(R) Platinum 8468 · Ubuntu 26.04.1 LTS · kernel 6.8.0-149-generic · driver 595.112.07
- **Run:** 2026-11-22T11:52:43Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 06:31 · benchmark 00:01 · whole run 06:50
- **Source:** run 427_20261122_115243
- **Command:** `bash run_benchmark.sh --smoke --model-name mistralai/Mistral-7B-v0.3 --dtype bfloat16 --kv-cache-dtype auto --gpu-memory-utilization 0.9 --max-model-len 128 --input-len 64 --output-len 16 --num-prompts 2 --concurrency 2 --seed 0`

| # | Metric | Result |
|---|---|---|
| 1 | Aggregate decode throughput, tokens/s (decode_throughput_tokens_sec) | 327.0740036253472 |
| 2 | Time To First Token p50, ms (ttft_msec) | 19.11804551491514 |
| 3 | TPOT p50, ms (tpot_msec) | 5.8107815323940795 |
| 4 | Peak KV-cache usage, GiB (kv_cache_peak_gb) | 0.019531908912005867 |
| 5 | KV-cache usage peak, pct (kv_cache_usage_peak_pct) | 0.03514197357323834 |
<!-- /profile:smoke -->
