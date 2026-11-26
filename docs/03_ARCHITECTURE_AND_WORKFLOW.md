# 03 — System Architecture & Workflow Plan

> **Purpose of this document:** Lay out the exact technical blueprint of how our testing system will work, explain the planned tools in simple words, and clarify how **Python / `liboqs`** and **Contiki-NG** work together cleanly without architectural dead-ends.

---

## 1. High-Level System Workflow (From Device Request to Comparison Report)

This flowchart defines the end-to-end engineering pipeline from cryptographic selection to visual analysis:

```mermaid
flowchart TD
    S1["1. IoT Client Node requests secure communication"] --> S2["2. Experiment Controller reads configs/ settings"]
    S2 --> S3{"3. Select Crypto Family"}

    S3 -->|"Old Baseline"| C1["Classical Engine: OpenSSL / Python cryptography - RSA & ECC"]
    S3 -->|"New Quantum-Safe"| P1["PQC Engine: liboqs / liboqs-python - ML-KEM & ML-DSA"]

    C1 --> S4["4. Execute Job: Key Sharing OR Digital Signature"]
    P1 --> S4

    S4 --> S5["5. Encrypt Message & Send over Virtual IoT Network"]
    S5 --> S6["6. Profiler Captures All 6 Metrics Across N Runs"]
    S6 --> S7["7. Save Structured Raw Results in data/ as CSV/JSON"]
    S7 --> S8["8. Matplotlib Analyzer generates Graphs & IoT Device Matrix in results/"]
```

---

## 2. Planned Tech Stack (Simple Tool-by-Tool Role)

| Tool | Plain-English Role | Why We Chose It |
| :--- | :--- | :--- |
| **Python 3** | Brain / coordinator that runs experiments, repeats tests, and logs timing/RAM | Easy to automate, readable across modules, rich tooling ecosystem |
| **OpenSSL / Python `cryptography`** | Provides trusted, industry-standard **RSA**, **ECDH**, and **ECDSA** | Fair, optimized baseline — avoids slow custom math scripts |
| **`liboqs` (`liboqs-python`)** | Open Quantum Safe library providing **ML-KEM** and **ML-DSA** | Official open-source C core with Python bindings supporting NIST FIPS 203 & 204 |
| **Contiki-NG (Cooja Simulator)** | Simulates constrained virtual IoT networks and low-power wireless packets | Lets us test IoT network conditions without buying dozens of hardware motes |
| **Linux Resource Controls / Docker / Optional Raspberry Pi** | Caps CPU/RAM or tests real ARM edge boards | Ensures software runs experience realistic IoT hardware limits |
| **Matplotlib (& Pandas)** | Turns raw CSV/JSON test logs into clear bar charts, boxplots, and trade-off graphs | Publication-ready figures for the research manuscript and summary reports |

---

## 3. Crucial Design Bridge: How `liboqs-python` and `Contiki-NG` Fit Together

A critical engineering question in this architecture is:
> *"Contiki-NG runs lightweight C code on tiny simulated microcontrollers, while `liboqs-python` runs in Python on Linux. How do those two tools connect?"*

Solving this upfront in our **Two-Layer Evaluation Architecture** ensures our implementation phase goes smoothly with zero surprises:

```mermaid
flowchart TD
    Plan["Unified Benchmark Study"] --> LayerA["Layer A: Compute & Memory Micro-Benchmark Engine"]
    Plan --> LayerB["Layer B: Virtual IoT Network & Fragmentation Simulator"]

    subgraph LA["Layer A - Host / Constrained Container / Optional RPi"]
        LA1["Python Runner + OpenSSL + liboqs-python"] --> LA2["Measures Exact Crypto Compute Time, Peak RAM, CPU %, Key & Signature Bytes"]
        LA2 --> LA3["Optional: Linux cgroups / Docker caps mimic IoT CPU & RAM limits"]
    end

    subgraph LB["Layer B - Contiki-NG / Cooja + Emulated Constrained Links"]
        LB1["Exact Byte Payloads Measured in Layer A feed Network Simulation"] --> LB2["Simulates IEEE 802.15.4 / 6LoWPAN or Throttled UDP/CoAP Links"]
        LB2 --> LB3["Measures Packet Fragmentation, Handshake Completion Time, Packet Loss Impact"]
    end

    LayerA --> LA1
    LayerB --> LB1
    LA3 --> Merge["Combined Analysis Engine"]
    LB3 --> Merge
    Merge --> Out["Final IoT Device-Tier Selection Guide & Research Paper Graphs"]
```

### Why This Two-Layer Design Is Smart
1. **Why not force all of `liboqs` + Python inside a tiny 32 KB Contiki mote?**
   `liboqs` is a general-purpose C library designed for desktop/server/SBC environments and does not fit unmodified inside ultra-tiny 10–32 KB microcontroller RAM, and Python cannot run on bare-metal sensor motes.
2. **How Layer A + Layer B give the FULL picture:**
   - **Layer A** gives **100% real, exact cryptographic execution numbers** (`RSA`, `ECC`, `ML-KEM`, `ML-DSA` via OpenSSL and `liboqs`) including compute latency, CPU %, RAM footprint, and exact serialized key/signature byte sizes. It can also be run on a constrained Docker container or a Raspberry Pi later!
   - **Layer B** uses **Contiki-NG / Cooja** (plus Linux `tc`/network namespaces if testing full IP handshakes) to show what happens when **those exact byte payloads** travel across a realistic, lossy, low-bandwidth IoT mesh network (packet fragmentation, airtime delay, handshake timeout).
   - *(Optional Stretch Goal in Phase 4/5)*: Test a lightweight C implementation (such as `PQClean` / `mupq` subset) directly in Contiki-NG's native/Cooja target for comparison!

---

## 4. Planned Data & Code Flow Across Repository Folders

```mermaid
flowchart LR
    CFG["configs/ - Test parameters, security tiers, trial counts"] --> RUN["src/benchmark/ - Automated test orchestrator"]
    CLS["src/classical/ - RSA, ECDH, ECDSA module"] --> RUN
    PQC["src/pqc/ - ML-KEM, ML-DSA module"] --> RUN
    NET["src/network_sim/ - Contiki-NG scripts & network emulator"] --> RUN
    RUN --> DATA["data/ - Raw & cleaned CSV/JSON benchmark logs"]
    DATA --> VIZ["results/ - Matplotlib charts & decision tables"]
    VIZ --> PAP["paper/ - Research paper figures & manuscript"]
```

---

## 5. Handshake Simulation State Flow (What One Benchmark Trial Looks Like)

Every single benchmark trial follows the same deterministic sequence to eliminate bias:

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> SetupEnv: Load algorithm & security level from config
    SetupEnv --> WarmUp: Run warm-up iterations (discarded from stats)
    WarmUp --> MeasureKeyGen: Start timer & memory tracker -> Generate Keypair
    MeasureKeyGen --> MeasureOp: Run Encaps/Decaps or Sign/Verify
    MeasureOp --> SimulateNetwork: Send exact bytes over virtual IoT link
    SimulateNetwork --> RecordMetrics: Log latency, CPU, RAM, key/sig bytes, total packets
    RecordMetrics --> CheckTrials: More trials remaining?
    CheckTrials --> MeasureKeyGen: Yes (repeat N times)
    CheckTrials --> ExportCSV: No (compute mean, median, std, p95)
    ExportCSV --> [*]
```
