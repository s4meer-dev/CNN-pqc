# `src/pqc/` — Post-Quantum Cryptography Module (Planned)

> **Status:** Blueprint only (Phase 3 deliverable).

## Planned Responsibility
Provide clean wrappers around **`liboqs` (`liboqs-python`)** for the standardized 2024 NIST Post-Quantum algorithms.

## Algorithms Planned Here
- **Key Sharing (Job 1 — FIPS 203):**
  - `ML-KEM-512` (NIST Level 1)
  - `ML-KEM-768` (NIST Level 3)
  - `ML-KEM-1024` (NIST Level 5)
- **Digital Signatures (Job 2 — FIPS 204):**
  - `ML-DSA-44` (NIST Category 2)
  - `ML-DSA-65` (NIST Category 3)
  - `ML-DSA-87` (NIST Category 5)

## Verification Check Planned
Every run will assert exact correctness (`client_shared_secret == server_shared_secret` for ML-KEM, and `verify(...) == True` for ML-DSA) so timings only count valid cryptographic exchanges.
