# Implementation Guide — Edge and Cloud Systems for ML Algorithms

This document describes how to implement the full project: **multiple ML workloads** using **both NeuroSim and SIAM** [1]. Tasks are **equally divided** among Parth, Lokesh, and Sachin (see README and sections below).

---

## 1. Prerequisites

- **NeuroSim v1.5** — cycle-accurate simulator for single-chip DNN accelerators ([arXiv:2505.02314](https://arxiv.org/abs/2505.02314))
- **SIAM** — chiplet-based scalable in-memory acceleration with mesh [1]; use **together with NeuroSim** for multi-chiplet / NoC/NoP scaling ([GitHub](https://github.com/gkrish19/SIAM), [arXiv:2107.02358](https://arxiv.org/abs/2107.02358))
- **Python** — for model export / integration (version as required by NeuroSim and SIAM)
- **Pre-trained models** — **VGG19**, **ViT-Base**, **ResNet-50** (and optionally **MobileNet**) — ImageNet-1K
- **Dataset** — ImageNet-1K validation subset (e.g. 1000 images across classes); CIFAR-10/100 if adding lighter workloads

---

## 2. Simulator Setup: NeuroSim + SIAM

We use **both** simulators; run the same workloads on each and compare.

### 2.1 NeuroSim (Parth)

1. Obtain NeuroSim v1.5 (repository or release from Georgia Tech / paper).
2. Install and build per documentation; run the provided example.
3. Note config/CLI for: buffer size, PE array, power, memory type, precision.

### 2.2 SIAM (Lokesh)

1. Obtain SIAM codebase ([GitHub](https://github.com/gkrish19/SIAM)) — chiplet-based IMC with mesh NoC/NoP [1].
2. Install and build per SIAM docs.
3. Run an example DNN; note how to set number of chiplets and mesh parameters.
4. Ensure the same workloads (VGG19, ResNet-50, ViT) can be run on both NeuroSim and SIAM.

---

## 3. Multiple ML Workloads (Model Integration)

Benchmark **at least three** DNNs so the study is not limited to a single workload.

### 3.1 VGG19 (Parth)

- Export VGG19 (138M params) to the format **both NeuroSim and SIAM** expect.
- Map layers to compute and memory model; run on 1000 validation images; log latency and energy.

### 3.2 ViT-Base (Lokesh)

- Export ViT-Base (86M params); handle patch embedding, attention, MLP.
- Integrate with **both NeuroSim and SIAM**; verify on same 1000-image subset and log metrics from each.

### 3.3 ResNet-50 (Sachin)

- Add ResNet-50 (≈25M params) as second CNN workload (different depth and structure than VGG19).
- Same pipeline: export for both NeuroSim and SIAM; run on 1000 validation images; log metrics.

### 3.4 Optional: MobileNet or another workload (team)

- Optional fourth workload (e.g. MobileNet) to broaden “multiple ML workloads”; split as agreed.

---

## 4. Using Both NeuroSim and SIAM (Packaging-Based Scaling)

We run the **same workloads on both** simulators:

- **NeuroSim:** Single-chip IMC; baseline latency, energy, area for each workload and config.
- **SIAM [1]:** Chiplet-based IMC with **mesh** NoC/NoP; run the same VGG19, ResNet-50, ViT with 1, 2, and 4 chiplets (or as supported) to study packaging-based scalability.
- **Comparison:** Table/figure comparing (e.g.) NeuroSim single-chip vs SIAM 1/2/4 chiplets for latency and energy; short discussion on when chiplet/mesh pays off.
- **Deliverable:** Report subsection “NeuroSim vs SIAM” with citation [1] and a comparison table or figure.

**Reference:** Krishnan, Gokul, et al. "SIAM: Chiplet-based scalable in-memory acceleration with mesh for deep neural networks." *ACM Trans. Embedded Comput. Syst. (TECS)* 20.5s (2021): 1–24.

---

## 5. Hardware Parameters to Sweep

| Parameter | Values | Owner |
|-----------|--------|--------|
| Buffer size | 4MB, 8MB, 16MB, 32MB | Parth |
| PE array | 64×64, 128×128, 256×256 | Parth |
| TDP (power) | 5W, 15W, 30W, 50W | Lokesh |
| Memory tech | SRAM, ReRAM | Lokesh |
| Precision | FP32, INT8 | Sachin |
| Chiplet/mesh (SIAM) | 1 / 2 / 4 chiplets or scaling sweep | Sachin |

Run each **workload** (VGG19, ResNet-50, ViT) per config. Log every run with a unique label (e.g. `resnet50_buf16_PE128_fp32`, `vgg19_siam_2chiplets`).

---

## 6. Metrics to Record

For each run, collect from **NeuroSim and (where applicable) SIAM**:

- **Latency** — inference time (ms)
- **Energy** — (mJ)
- **Area** — (mm²) if reported
- **Peak memory bandwidth utilization** — (%) if reported
- **GOPS/W** — if reported

Compute **EDP = Energy × Latency** for each run (Sachin: spreadsheet or script).

---

## 7. Running Experiments

1. **Single run:** Set one combination of buffer, PE, TDP, memory tech, precision (and chiplet count if using SIAM); run for **each workload** (VGG19, ResNet-50, ViT) on the 1000-image set; save stdout/log per config.
2. **Sweeps:** Repeat for all combinations; ensure **multiple workloads** are covered, not just one DNN.
3. **Logging:** Use a table (CSV or sheet) with columns: **simulator** (NeuroSim / SIAM), **workload**, buffer_MB, PE, TDP_W, memory_tech, precision, [chiplets for SIAM], latency_ms, energy_mJ, EDP, notes.

---

## 8. Results and Report

- **Parth:** Multi-workload **latency** table — rows = configs, columns = PE sizes; one block per workload (VGG19, ResNet-50, ViT).
- **Lokesh:** **Energy / area** summary; **NeuroSim vs SIAM** comparison (table or figure + short discussion).
- **Sachin:** **Quantization (FP32 vs INT8)** and **EDP/utilization** table; **optimal config** (1–2 sentences); consolidate all result tables and **draft final report**.

---

## 9. Suggested Folder Layout

```
Edge_project/
├── README.md
├── IMPLEMENTATION.md          # this file
├── Project_Edge_and_cloud_systems_for_ML_Algo.pdf
├── neurosim/                  # NeuroSim code/configs
├── siam/                      # SIAM code/configs (use together with NeuroSim)
├── models/                    # export scripts / weights for VGG19, ResNet-50, ViT (multi-workload)
├── scripts/                   # run_sweep.sh, parse_results.py, etc.
├── results/                   # logs, CSV, tables
└── report/                    # short final report
```

---

## 10. Checklist

- [ ] NeuroSim built and example run OK  
- [ ] SIAM built and example run OK (use **both** NeuroSim and SIAM)  
- [ ] VGG19 + ResNet-50 integrated; one full run each on **NeuroSim and SIAM** logged  
- [ ] ViT integrated; one full run each on NeuroSim and SIAM logged  
- [ ] Buffer + PE sweep done for **all workloads** (VGG19, ResNet-50, ViT)  
- [ ] TDP + memory tech sweep done  
- [ ] FP32 vs INT8 runs done across workloads  
- [ ] All metrics in one table; EDP computed; multi-workload comparison done  
- [ ] NeuroSim vs SIAM comparison (table/figure) and citation [1] in report  
- [ ] Short report drafted  

Use this as the single implementation reference for the whole project; update it as you fix paths, tool versions, or add scripts.
