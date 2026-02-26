# NeuroSim Beginner's Guide

## What is NeuroSim?

NeuroSim is a **hardware simulator** for evaluating DNN (Deep Neural Network) accelerators at the circuit level. It estimates:
- **Latency** (how long inference takes)
- **Energy** consumption
- **Area** (chip size)
- Hardware effects (ADC precision, memory cell behavior, etc.)

---

## Quick Start: Running Your First Example

### 1. Basic Command Structure

```bash
cd /home/parthpatel/NeuroSim/Inference_pytorch

python3 inference.py \
  --dataset cifar10 \
  --model VGG8 \
  --mode WAGE \
  --inference 1 \
  --cellBit 1 \
  --subArray 128 \
  --parallelRead 64
```

### 2. What Each Parameter Means

| Parameter | What it does | Common values |
|-----------|--------------|---------------|
| `--dataset` | Dataset to use | `cifar10`, `cifar100`, `imagenet` |
| `--model` | Neural network model | `VGG8`, `DenseNet40`, `ResNet18` |
| `--mode` | Precision mode | `WAGE` (8-bit), `FP` (floating point) |
| `--inference` | Enable hardware simulation | `1` (yes), `0` (no - software only) |
| `--cellBit` | Bits per memory cell | `1`, `2`, `4` |
| `--subArray` | PE array size | `64`, `128`, `256` |
| `--parallelRead` | Rows read in parallel | `32`, `64`, `128` (≤ subArray) |
| `--ADCprecision` | ADC bit precision | `4`, `5`, `6` |
| `--onoffratio` | Device on/off ratio | `10`, `100` |
| `--batch_size` | Images per batch | `200`, `500` (larger = faster) |

---

## Available Pre-trained Models

NeuroSim includes these ready-to-use models:

1. **VGG8** on CIFAR10 (8-bit WAGE mode)
   - Location: `log/VGG8.pth`
   - Fast, good for testing

2. **DenseNet40** on CIFAR10 (8-bit WAGE mode)
   - Location: `log/DenseNet40.pth`
   - Medium complexity

3. **ResNet18** on ImageNet (FP mode)
   - Downloads automatically from PyTorch
   - Larger, takes longer

---

## Step-by-Step: Running VGG8 Example

```bash
# 1. Go to the inference directory
cd /home/parthpatel/NeuroSim/Inference_pytorch

# 2. Run the example (uses GPU automatically if available)
python3 inference.py \
  --dataset cifar10 \
  --model VGG8 \
  --mode WAGE \
  --inference 1 \
  --cellBit 1 \
  --subArray 128 \
  --parallelRead 64

# 3. Wait for completion (5-15 minutes)
# You'll see: layer quantization → NeuroSim simulation → results

# 4. Check results
# Results are in: log/default/ADCprecision=5/batch_size=200/.../log.txt
```

---

## Understanding the Hardware Parameters

### Buffer Size (Configured in C++ code - Param.cpp)
- Controls on-chip memory
- Values: 4 MB, 8 MB, 16 MB, 32 MB
- Larger = fewer memory accesses but bigger chip

### PE (Processing Element) Array Size
- Set with `--subArray`
- Values: 64×64, 128×128, 256×256
- Larger = more parallel computation, lower latency

### Memory Cell Precision
- Set with `--cellBit`
- Values: 1-bit, 2-bit, 4-bit
- Higher = better accuracy but more complex hardware

### Parallel Read
- Set with `--parallelRead`
- Must be ≤ subArray size
- Higher = more parallelism but more power

---

## Typical Use Cases

### Case 1: Quick Test (Fast)
```bash
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE --inference 1 \
  --cellBit 1 --subArray 128 --parallelRead 64
```
**Time: ~5-10 minutes**

### Case 2: Higher Precision
```bash
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE --inference 1 \
  --cellBit 4 --subArray 128 --parallelRead 64 --ADCprecision 6
```
**Time: ~10-15 minutes**

### Case 3: Larger PE Array (Faster Hardware)
```bash
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE --inference 1 \
  --cellBit 1 --subArray 256 --parallelRead 128
```
**Time: ~5-10 minutes**

### Case 4: ImageNet with ResNet18 (Large)
```bash
python3 inference.py --dataset imagenet --model ResNet18 --mode FP --inference 1 \
  --subArray 128 --parallelRead 64 --onoffratio 100
```
**Time: ~30-60 minutes**

---

## Where to Find Results

After running, results are saved in:
```
log/default/ADCprecision=X/batch_size=Y/.../log.txt
```

Key metrics in the output:
- **Latency** (ms): Total inference time
- **Energy** (mJ): Total energy consumption
- **Area** (mm²): Chip area
- **Throughput** (GOPS): Operations per second
- **Accuracy**: Classification accuracy

---

## Common Issues & Solutions

### Issue 1: "CUDA out of memory"
**Solution:** Reduce batch size
```bash
--batch_size 100  # Instead of 500
```

### Issue 2: "Dataset not found"
**Solution:** Data downloads automatically, wait for it to complete

### Issue 3: Process seems stuck
**Solution:** It's not stuck! NeuroSim does intensive circuit simulations
- Check GPU usage: `nvidia-smi`
- Should show 90-100% GPU utilization
- First run takes longer (model loading + quantization)


---

## Useful Commands

```bash
# Check if NeuroSim is running
ps aux | grep inference.py

# Check GPU usage
nvidia-smi

# Stop a running inference
pkill -f inference.py

# Check results directory
ls -lh log/default/

# Monitor real-time GPU usage
watch -n 1 nvidia-smi
```

---

## Need Help?

- **Manual**: `/home/parthpatel/NeuroSim/Documents/DNN NeuroSim V1.4 Manual.pdf`
- **README**: `/home/parthpatel/NeuroSim/README.md`
- **Example networks**: `/home/parthpatel/NeuroSim/Inference_pytorch/NeuroSIM/NetWork_*.csv`
- **Your plan**: `/home/parthpatel/Project_Edge_and_cloud_systems_for_ML_Algo/parth.md`


