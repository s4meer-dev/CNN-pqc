# `data/` — Raw & Processed Benchmark Logs (Planned)

> **Status:** Empty until Phase 5 experiments begin.

## Purpose
Store genuine, timestamped CSV / JSON output files generated automatically by `src/benchmark/`.

## Planned Organization
- `data/raw/` — Per-trial measurements (`timestamp`, `algorithm`, `job`, `security_level`, `key_bytes`, `sig_or_ct_bytes`, `latency_ms`, `cpu_ms`, `peak_ram_kb`, `overhead_bytes`)
- `data/processed/` — Aggregated statistics (`mean`, `median`, `std_dev`, `min`, `max`, `p95`) used directly by `results/` plotting scripts and `paper/` tables
