# Edge and Cloud Systems for ML Algorithms

**Team:** Parth, Lokesh, Sachin  
**Timeline:** 1 month  
**Date:** February 2026

---

## What We’re Doing

We benchmark **multiple ML workloads** (not just a single DNN) using **both NeuroSim and SIAM** [1].

- **Workloads:** **VGG19**, **ViT-Base**, and **ResNet-50** (and optionally **MobileNet**) — covering CNNs, vision transformers, and different scales.
- **Simulation (both tools):**
  - **NeuroSim** v1.5 — single-chip in-memory acceleration; device/circuit/architecture-level metrics (latency, energy, area).
  - **SIAM** [1] — chiplet-based scalable in-memory acceleration with **mesh** NoC/NoP; multi-chiplet scaling and packaging.
- We run the **same workloads on both** NeuroSim and SIAM and compare single-chip vs chiplet/mesh scaling.
- **Hardware knobs:** Buffer size, PE array, power (TDP), memory (SRAM/ReRAM), precision (FP32/INT8), and **chiplet count / mesh** in SIAM.

---

## Timeline (1 Month)

| Week | Focus |
|------|--------|
| **Week 1** | NeuroSim setup; VGG19 + ResNet-50 running |
| **Week 2** | ViT running; **SIAM** setup and integration (with NeuroSim) |
| **Week 3** | Buffer, PE, power, memory tech, quantization across all workloads |
| **Week 4** | Multi-workload comparison, tables, short report |

---

## Work Split

| Area | Parth | Lokesh | Sachin |
|------|--------|--------|--------|
| **Simulator** | NeuroSim setup | SIAM setup | — |
| **Model** | VGG19 (NeuroSim + SIAM) | ViT (NeuroSim + SIAM) | ResNet-50 (NeuroSim + SIAM) |
| **Param sweeps** | Buffer size; PE array | TDP; Memory (SRAM/ReRAM) | Precision (FP32/INT8); Chiplet/mesh |
| **Results** | Latency tables (all workloads) | Energy/area; NeuroSim vs SIAM | EDP & quantization; result tables; report |

**Parth** — NeuroSim setup; VGG19 on both tools; buffer + PE sweeps; latency tables.  
**Lokesh** — SIAM setup; ViT on both tools; TDP + memory-tech sweeps; energy/area + NeuroSim vs SIAM comparison.  
**Sachin** — ResNet-50 on both tools; precision + chiplet sweeps; EDP/quantization summary; consolidate result tables and draft final report.

---

## References

- [1] G. Krishnan et al., "SIAM: Chiplet-based scalable in-memory acceleration with mesh for deep neural networks," *ACM Trans. Embedded Comput. Syst. (TECS)*, 20(5s), 2021, pp. 1–24. [arXiv:2107.02358](https://arxiv.org/abs/2107.02358)
- NeuroSim V1.5: [arXiv:2505.02314](https://arxiv.org/abs/2505.02314)  
- VGG19: [arXiv:1409.1556](https://arxiv.org/abs/1409.1556)  
- ViT: [arXiv:2010.11929](https://arxiv.org/abs/2010.11929)
