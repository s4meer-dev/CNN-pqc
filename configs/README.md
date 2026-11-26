# `configs/` — Experiment Configuration Blueprints (Planned)

> **Status:** Blueprint only (Phase 3–4 deliverable).

## Purpose
Keep every experiment parameter **outside the code** in readable YAML or JSON files so tests stay reproducible and fair across different test environments.

## Planned Configuration Parameters
- **Security Tiers:** Matched Level 1 (~128-bit), Level 3 (~192-bit), Level 5 (~256-bit) algorithm pairs
- **Message / Payload Sizes:** `64 B`, `256 B`, `1024 B`
- **Trial Settings:** `warmup_runs: 5`, `iterations: 50` (or `100`)
- **Network Profiles:** Ideal local link, 250 kbps IoT mesh (`802.15.4`-like), constrained lossy link
