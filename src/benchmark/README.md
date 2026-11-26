# `src/benchmark/` — Automated Runner, Profiler & Plotter (Planned)

> **Status:** Blueprint only (Phase 4–5 deliverable).

## Planned Responsibility
Automate repeated, controlled experiments across both Classical and PQC algorithms and record all **6 core metrics**.

## Planned Scripts
- **`runner.py` (Planned):** Reads experiment definitions from `../../configs/`, executes warm-up runs + `N` timed trials, and writes structured logs into `../../data/`.
- **`profiler.py` (Planned):** Captures high-resolution latency (`time.perf_counter_ns()`), CPU process time, peak RAM (`tracemalloc` + RSS), and serialized payload sizes.
- **`plotter.py` (Planned):** Reads `../../data/` CSV files and generates Matplotlib comparison charts into `../../results/`.
