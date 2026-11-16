# Results: 427 KV Cache Stress Test

**Main goal:** Analyze LLM KV-cache stress under long contexts  
**Tools:** Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_142 · recorded 2026-11-16). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 NVL (95830 MiB) · AMD EPYC 9374F 32-Core Processor · Ubuntu 26.04.1 LTS · kernel 6.8.0-138-generic · driver 595.91.07
- **Run:** 2026-11-16T11:37:51Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 03:44 · benchmark 00:01 · whole run 03:59
- **Source:** run 427_20261116_113751
- **Command:** `bash run_benchmark.sh --smoke --model-name mistralai/Mistral-7B-v0.3 --dtype bfloat16 --kv-cache-dtype auto --gpu-memory-utilization 0.9 --max-model-len 128 --input-len 64 --output-len 16 --num-prompts 2 --concurrency 2 --seed 0`

| # | Metric | Result |
|---|---|---|
| 1 | Aggregate decode throughput, tokens/s (decode_throughput_tokens_sec) | 342.5068400867448 |
| 2 | Time To First Token p50, ms (ttft_msec) | 25.335745187476277 |
| 3 | TPOT p50, ms (tpot_msec) | 5.619025820245346 |
| 4 | Peak KV-cache usage, GiB (kv_cache_peak_gb) | 0.023437042984543174 |
| 5 | KV-cache usage peak, pct (kv_cache_usage_peak_pct) | 0.03441057551687443 |
<!-- /profile:smoke -->
