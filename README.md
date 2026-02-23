# Edge and Cloud Systems for ML Algorithms

**Team:** Parth, Lokesh, Sachin  
**Timeline:** 1 month  
**Date:** February 2026

---

## What We’re Doing

We use **NeuroSim** to see how different hardware settings affect inference for **VGG19** and **Vision Transformer (ViT)** on edge-style systems. We change buffer size, PE array size, power, memory type (SRAM/ReRAM), and precision (FP32/INT8), and look at latency, energy, and area.

---

## Timeline (1 Month)

| Week | Focus |
|------|--------|
| **Week 1** | NeuroSim setup, get VGG19 running |
| **Week 2** | ViT running, buffer & PE experiments |
| **Week 3** | Power, memory tech, quantization runs |
| **Week 4** | Results, tables, short report |

---

## Work Split

**Parth (Project Lead)**  
- Overall coordination and deadlines  
- NeuroSim setup + VGG19 integration  
- Buffer & PE configs, latency numbers  

**Lokesh**  
- ViT integration with NeuroSim  
- Power (5W–50W) and SRAM vs ReRAM runs  
- Energy and area summary  

**Sachin**  
- FP32 vs INT8 experiments  
- EDP and utilization summary  
- Result tables + final report draft  

---

## References

- NeuroSim V1.5: [arXiv:2505.02314](https://arxiv.org/abs/2505.02314)  
- VGG19: [arXiv:1409.1556](https://arxiv.org/abs/1409.1556)  
- ViT: [arXiv:2010.11929](https://arxiv.org/abs/2010.11929)
