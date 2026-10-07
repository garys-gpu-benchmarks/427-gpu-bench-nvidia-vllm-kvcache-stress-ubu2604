# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, num_prompts, concurrency, and max_model_len come from yaml. output_format: csv Sweep dimensions: serving_engine, model_name, dtype, kv_cache_dtype, gpu_memory_utilization, max_model_len, dataset_name, input_len.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| serving_engine | `--serving-engine` | smoke=vllm, baseline=vllm, extended=vllm | vllm | From Parameter list; see Execution Description With Parameters. |
| model_name | `--model-name` | smoke=mistralai/Mistral-7B-v0.3, baseline=mistralai/Mistral-7B-v0.3, extended=mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 | From Parameter list; see Execution Description With Parameters. |
| dtype | `--dtype` | smoke=bfloat16, baseline=bfloat16, extended=bfloat16 | bfloat16 | From Parameter list; see Execution Description With Parameters. |
| kv_cache_dtype | `--kv-cache-dtype` | smoke=auto, baseline=auto, extended=auto | auto | From Parameter list; see Execution Description With Parameters. |
| gpu_memory_utilization | `--gpu-memory-utilization` | smoke=0.9, baseline=0.9, extended=0.9 | 0.9 | From Parameter list; see Execution Description With Parameters. |
| max_model_len | `--max-model-len` | smoke=128, baseline=8192, extended=8192 | 8192 | From Parameter list; see Execution Description With Parameters. |
| dataset_name | `--dataset-name` | smoke=synthetic, baseline=synthetic, extended=synthetic | synthetic | From Parameter list; see Execution Description With Parameters. |
| input_len | `--input-len` | smoke=64, baseline=4096, extended=4096 | 4096 | From Parameter list; see Execution Description With Parameters. |
| output_len | `--output-len` | smoke=16, baseline=256, extended=256 | 256 | From Parameter list; see Execution Description With Parameters. |
| num_prompts | `--num-prompts` | smoke=2, baseline=136, extended=768 | 136 | From Parameter list; see Execution Description With Parameters. |
| concurrency | `--concurrency` | smoke=2, baseline=8, extended=8 | 8 | From Parameter list; see Execution Description With Parameters. |
| seed | `--seed` | smoke=0, baseline=0, extended=0 | 0 | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py
```

## Raw Output Format

raw_results.csv with sweep_0, sweep_1, and a summary row. raw_results.jsonl and raw_output.txt carry the same records

check_name,status,completed_requests,output_tokens_per_sec,ttft_msec,tpot_msec,kv_cache_usage_peak_pct,kv_cache_peak_gb,sustained_decode_throughput_under_cache_pressure_tokens_sec
sweep_0,ok,8,40,25,12,20,8,40
summary,ok,8,40,25,12,20,8,40

## Metrics

- **#1: Decode throughput, tokens/s** — stored as `sustained_decode_throughput_under_cache_pressure_tokens_sec`.
- **#2: Time To First Token, ms** — stored as `ttft_msec`.
- **#3: TPOT, ms** — stored as `tpot_msec`.
- **#4: Peak KV-cache usage, GiB** — stored as `kv_cache_peak_gb`.
- **#5: KV-cache usage peak, pct** — stored as `kv_cache_usage_peak_pct`.

## Framework

Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, num_prompts, concurrency, and max_model_len come from yaml. output_format: csv

## Installation and Execution Summary

Start python -m vllm.entrypoints.openai.api_server --model mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, wait for /v1/models, run scripts/benchmark_serving.py once with yaml input_len, output_len, num_prompts, and concurrency, then stop the server, to measure KV-cache stress

## Platform Portability

- **AMD (primary):** ```bash
Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py
```
- **NVIDIA:** Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

raw_results.csv with sweep_0, sweep_1, and a summary row. raw_results.jsonl and raw_output.txt carry the same records

check_name,status,completed_requests,output_tokens_per_sec,ttft_msec,tpot_msec,kv_cache_usage_peak_pct,kv_cache_peak_gb,sustained_decode_throughput_under_cache_pressure_tokens_sec
sweep_0,ok,8,40,25,12,20,8,40
summary,ok,8,40,25,12,20,8,40

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, num_prompts, concurrency, and max_model_len come from yaml. output_format: csv
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, num_prompts, concurrency, and max_model_len come from yaml. output_format: csv

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
