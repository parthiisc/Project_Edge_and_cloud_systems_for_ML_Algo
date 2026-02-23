# Implementation Guide — Edge and Cloud Systems for ML Algorithms

This document describes how to implement the full project: setup, experiments, and results.

---

## 1. Prerequisites

- **NeuroSim v1.5** — cycle-accurate simulator for DNN accelerators ([arXiv:2505.02314](https://arxiv.org/abs/2505.02314))
- **Python** — for model export / integration (version as required by NeuroSim)
- **Pre-trained models** — VGG19 and ViT-Base (e.g. from PyTorch/TensorFlow, ImageNet-1K)
- **Dataset** — ImageNet-1K validation subset (e.g. 1000 images across classes) for evaluation

---

## 2. NeuroSim Setup (Parth)

1. Obtain NeuroSim v1.5 (repository or release from Georgia Tech / paper).
2. Install dependencies per NeuroSim documentation (compiler, any Python/API requirements).
3. Build the simulator and run the provided example to confirm it runs.
4. Note the config file or CLI options used for: buffer size, PE array, power, memory type, precision.

---

## 3. Model Integration

### 3.1 VGG19 (Parth)

- Export VGG19 (138M params) to the format NeuroSim expects (e.g. layer list, dimensions, ops).
- Map layers to NeuroSim’s compute and memory model.
- Run inference on 1000 validation images and confirm NeuroSim reports latency and energy for at least one fixed hardware config.

### 3.2 ViT-Base (Lokesh)

- Export ViT-Base (86M params) similarly (patch embedding, attention blocks, MLP).
- Integrate with NeuroSim; handle attention and non-convolutional ops as required by the tool.
- Verify runs on the same 1000-image subset and that metrics are logged.

---

## 4. Hardware Parameters to Sweep

| Parameter | Values | Owner |
|-----------|--------|--------|
| Buffer size | 4MB, 8MB, 16MB, 32MB | Parth |
| PE array | 64×64, 128×128, 256×256 | Parth |
| TDP (power) | 5W, 15W, 30W, 50W | Lokesh |
| Memory tech | SRAM, ReRAM | Lokesh |
| Precision | FP32, INT8 | Sachin |

Run one model (VGG19 or ViT) per config; repeat for the other model. Log every run with a unique label (e.g. `vgg19_buf16_PE128_fp32`).

---

## 5. Metrics to Record

For each run, collect from NeuroSim:

- **Latency** — inference time (ms)
- **Energy** — (mJ)
- **Area** — (mm²) if reported
- **Peak memory bandwidth utilization** — (%) if reported
- **GOPS/W** — if reported

Compute **EDP = Energy × Latency** for each run (Sachin can do this in a spreadsheet or script).

---

## 6. Running Experiments

1. **Single run:** Set one combination of buffer, PE, TDP, memory tech, precision; run NeuroSim for VGG19 (or ViT) on the 1000-image set; save stdout/log to a file named by config.
2. **Sweeps:** Repeat for all combinations you need (full sweep or a reduced set as agreed).
3. **Logging:** Use a simple table (CSV or Google Sheet) with columns: model, buffer_MB, PE, TDP_W, memory_tech, precision, latency_ms, energy_mJ, EDP, notes.

---

## 7. Results and Report

- **Latency table** — e.g. rows = configs, columns = 64×64 / 128×128 / 256×256 for VGG19 and ViT (Parth).
- **Energy / area** — summary and short notes (Lokesh).
- **Quantization (FP32 vs INT8)** and **EDP/utilization** — table and short summary (Sachin).
- **Optimal config** — one recommended setting (e.g. 15W edge: 128×128 PE, 16MB buffer, INT8, ReRAM) and 1–2 sentences justification.

---

## 8. Suggested Folder Layout

```
Edge_project/
├── README.md
├── IMPLEMENTATION.md          # this file
├── Project_Edge_and_cloud_systems_for_ML_Algo.pdf
├── neurosim/                  # NeuroSim code/configs (or link to it)
├── models/                    # export scripts / weights info for VGG19, ViT
├── scripts/                   # run_sweep.sh, parse_results.py, etc.
├── results/                   # logs, CSV, tables
└── report/                    # short final report
```

---

## 9. Checklist

- [ ] NeuroSim built and example run OK  
- [ ] VGG19 integrated and one full run logged  
- [ ] ViT integrated and one full run logged  
- [ ] Buffer + PE sweep done for both models  
- [ ] TDP + memory tech sweep done  
- [ ] FP32 vs INT8 runs done  
- [ ] All metrics in one table; EDP computed  
- [ ] Latency/energy/EDP summary and optimal config written  
- [ ] Short report drafted  

Use this as the single implementation reference for the whole project; update it as you fix paths, tool versions, or add scripts.
