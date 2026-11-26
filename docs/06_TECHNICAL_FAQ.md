# 06 — Technical FAQ & Key Design Decisions

> **Purpose of this document:** Clarify common technical questions about Post-Quantum Cryptography in IoT and explain the rationale behind key architectural choices in simple, direct language.

---

## Part A — Core Concept Questions

### Q1: What is the main objective of this project in one sentence?
**Answer:** Evaluate newly standardized post-quantum cryptographic algorithms (`ML-KEM` and `ML-DSA`) against classical algorithms (`RSA` and `ECC`) inside a simulated IoT environment to determine which quantum-safe mechanisms fit different IoT resource tiers.

### Q2: Can quantum computers break `RSA` and `ECC` today? Why prepare now?
**Answer:** Large-scale fault-tolerant quantum computers capable of breaking `RSA-2048` do not exist publicly yet. However, adversaries can capture and store encrypted IoT traffic today and decrypt it years later once quantum capability matures (*"Harvest Now, Decrypt Later"*). Because IoT deployments often operate for 10+ years, migration planning is needed now.

### Q3: Does running Post-Quantum Cryptography require quantum hardware?
**Answer:** **No.** PQC algorithms are standard software algorithms running on ordinary CPUs and microcontrollers. They are called *post-quantum* because their underlying mathematics (such as module lattices) resists known quantum attacks like Shor's algorithm.

### Q4: Why is PQC especially challenging for IoT devices?
**Answer:** IoT devices operate under tight RAM limits (sometimes tens of kilobytes), low clock-speed processors, strict battery constraints, and small wireless packet payloads (e.g., 127-byte IEEE 802.15.4 frames). Because `ML-KEM` and `ML-DSA` keys and signatures are substantially larger than `ECC` keys and signatures, they increase memory pressure, packet fragmentation, and transmission energy.

### Q5: Why compare same-job algorithms only?
**Answer:** Key encapsulation/exchange (`RSA` key transport, `ECDH`, `ML-KEM`) and digital signatures (`RSA` sign, `ECDSA`, `ML-DSA`) serve completely different cryptographic roles in a handshake. Comparing them across jobs would misrepresent performance; every comparison in this project pairs equivalent roles at matched security levels.

### Q6: Why select `ML-KEM` and `ML-DSA` specifically?
**Answer:** In August 2024, NIST finalized `ML-KEM` (**FIPS 203**, derived from CRYSTALS-Kyber) for key encapsulation and `ML-DSA` (**FIPS 204**, derived from CRYSTALS-Dilithium) for digital signatures as primary global standards, both supported in `liboqs`.

---

## Part B — Architecture & Methodology Questions

### Q7: Why use a simulated IoT network instead of only physical hardware?
**Answer:** A virtual simulation environment enables reproducible, controlled experiments across identical bandwidth caps, packet-loss rates, and multi-hop topologies without hardware variability or procurement cost. Validated baselines can later be cross-checked on edge boards such as a Raspberry Pi.

### Q8: How is experimental fairness enforced?
**Answer:** All comparisons match equivalent NIST security strengths (e.g., ~128-bit and ~192-bit levels), run under identical container/host resource limits, send identical payload sizes, discard warm-up runs, and average over 50–100 repeated trials with median, mean, standard deviation, and `p95` reporting.

### Q9: Why not simply increase `RSA` key sizes (e.g., `RSA-8192`) instead of adopting PQC?
**Answer:** Increasing classical modulus sizes only raises classical cost while remaining vulnerable to **Shor's algorithm** in polynomial time on a quantum computer — and `RSA-8192` is already prohibitively slow for constrained IoT processors.

### Q10: How do Python/`liboqs-python` and C-based `Contiki-NG` fit together?
**Answer:** The project uses a **Two-Layer Evaluation Architecture**:
- **Layer A** runs exact `OpenSSL` and `liboqs-python` implementations under controlled CPU/RAM quotas to capture genuine execution latency, CPU time, peak RAM, and serialized key/signature byte lengths.
- **Layer B** feeds those exact payload byte sizes into **Contiki-NG / Cooja** (plus constrained virtual links) to measure 6LoWPAN/802.15.4 packet fragmentation, airtime overhead, and loss resilience.

### Q11: What is the difference between Pure PQC and Hybrid Cryptography?
**Answer:** **Pure PQC** relies exclusively on a post-quantum algorithm (e.g., `ML-KEM`). **Hybrid cryptography** pairs a classical primitive with a PQC primitive (e.g., `ECC + ML-KEM`) so security holds as long as *either* primitive remains unbroken, trading extra bytes and CPU cycles for transition resilience.

### Q12: Have benchmark measurements been generated yet?
**Answer:** **No.** The project is currently in **Phase 1 (Planning & Architecture)**; no benchmark scripts or measurements exist yet, and any expected trends in the plan are explicitly marked as pre-experiment hypotheses.
