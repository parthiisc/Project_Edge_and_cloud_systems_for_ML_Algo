# NeuroSim Configuration Guide

**For Project: Edge and Cloud Systems for ML Algorithms**  
**Author: Parth**  
**Date: February 26, 2026**

---

## Table of Contents

1. [Command-Line Parameters](#command-line-parameters)
2. [Hardware Configuration (Param.cpp)](#hardware-configuration-paramcpp)
3. [Buffer Configuration](#buffer-configuration)
4. [PE Array Configuration](#pe-array-configuration)
5. [Technology Node Selection](#technology-node-selection)
6. [Memory Cell Types](#memory-cell-types)
7. [Quick Config Cheat Sheet](#quick-config-cheat-sheet)

---

## Command-Line Parameters

These are set when running `inference.py` from the command line:

### Basic Parameters

```bash
python3 inference.py \
  --dataset cifar10 \          # Dataset: cifar10 | cifar100 | imagenet
  --model VGG8 \               # Model: VGG8 | DenseNet40 | ResNet18 | VGG19
  --mode WAGE \                # Precision: WAGE (8-bit) | FP (floating point)
  --batch_size 200 \           # Batch size (larger = faster but more memory)
  --inference 1                # 1=hardware simulation, 0=software only
```

### Hardware Simulation Parameters

| Parameter | Description | Values | Default | Your Sweeps |
|-----------|-------------|--------|---------|-------------|
| `--inference` | Enable HW simulation | `0` (off), `1` (on) | `0` | Always `1` |
| `--subArray` | PE array size | `64`, `128`, `256` | `128` | **✓ Sweep** |
| `--parallelRead` | Parallel read rows | `32`, `64`, `128`, `256` | `128` | Match subArray |
| `--cellBit` | Bits per cell | `1`, `2`, `4` | `1` | `1` (baseline) |
| `--ADCprecision` | ADC bit precision | `4`, `5`, `6`, `7`, `8` | `5` | `5` (baseline) |
| `--onoffratio` | Device On/Off ratio | `10`, `100`, `1000` | `10` | `10` (baseline) |

### Device Variability Parameters (Advanced)

| Parameter | Description | Values | Default |
|-----------|-------------|--------|---------|
| `--vari` | Conductance variation | `0.0` - `1.0` | `0.0` |
| `--t` | Retention time | `0` - `∞` | `0` |
| `--v` | Drift coefficient | `0.0` - `1.0` | `0.0` |
| `--detect` | Drift direction | `0` (random), `1` (fixed) | `0` |

### Example Commands

**Baseline (for your sweeps):**
```bash
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE \
  --inference 1 --cellBit 1 --ADCprecision 5 --onoffratio 10 \
  --subArray 128 --parallelRead 64
```

**PE Array Sweep:**
```bash
# 64×64
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE \
  --inference 1 --subArray 64 --parallelRead 64

# 128×128
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE \
  --inference 1 --subArray 128 --parallelRead 64

# 256×256
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE \
  --inference 1 --subArray 256 --parallelRead 128
```

---

## Hardware Configuration (Param.cpp)

**Location:** `/home/parthpatel/NeuroSim/Inference_pytorch/NeuroSIM/Param.cpp`

These parameters require **modifying the C++ code** and **recompiling**:

```bash
cd /home/parthpatel/NeuroSim/Inference_pytorch/NeuroSIM
# Edit Param.cpp
nano Param.cpp   # or use any editor
# Recompile
make clean && make
```

### Key Parameters in Param.cpp

#### Line 56-58: Operation Mode
```cpp
operationmode = 2;     // 1: conventionalSequential
                       // 2: conventionalParallel (DEFAULT)
```

#### Line 61-63: Memory Cell Type
```cpp
memcelltype = 1;       // 1: SRAM (DEFAULT)
                       // 2: RRAM
                       // 3: FeFET
```

#### Line 72-73: Device Roadmap
```cpp
deviceroadmap = 2;     // 1: HP (High Performance)
                       // 2: LSTP (Low Standby Power) (DEFAULT)
```

#### Line 76-77: Global Bus Type
```cpp
globalBusType = false; // false: X-Y Bus (DEFAULT)
                       // true: H-Tree
```

#### Line 79-82: Global Buffer Configuration
```cpp
globalBufferType = false;      // false: register file (DEFAULT)
                               // true: SRAM
globalBufferCoreSizeRow = 128; // Rows in global buffer
globalBufferCoreSizeCol = 128; // Columns in global buffer
```

**Buffer Size Calculation:**
- Register file: `globalBufferCoreSizeRow × globalBufferCoreSizeCol × word_size`
- For 128×128 with 8-bit: ~16 KB (small)
- You'll need to increase these for MB-level buffers

#### Line 84-87: Tile Buffer Configuration
```cpp
tileBufferType = false;       // false: register file (DEFAULT)
                              // true: SRAM
tileBufferCoreSizeRow = 32;   // Rows in tile buffer
tileBufferCoreSizeCol = 32;   // Columns in tile buffer
```

#### Line 89-90: PE Buffer Configuration
```cpp
peBufferType = false;         // false: register file (DEFAULT)
                              // true: SRAM
```

#### Line 125-127: Operating Conditions
```cpp
clkFreq = 1e9;               // Clock frequency (1 GHz)
temp = 300;                  // Temperature (K) = 27°C
technode = 22;               // Technology node (nm)
```

#### Line 203-204: SubArray Size
```cpp
numRowSubArray = 128;        // Rows in subArray (PE array)
numColSubArray = 128;        // Columns in subArray (PE array)
```

**Note:** These are **overridden** by `--subArray` command-line parameter

---

## Buffer Configuration

### For Your Buffer Sweeps (4, 8, 16, 32 MB)

You need to modify **Param.cpp** to change buffer sizes. Here's how:

#### Step 1: Calculate Buffer Dimensions

Target buffer sizes and corresponding dimensions (assuming 8-bit words):

| Target Size | Rows | Cols | Calculation |
|-------------|------|------|-------------|
| 4 MB | 2048 | 2048 | 2048 × 2048 × 1 byte = 4 MB |
| 8 MB | 2896 | 2896 | 2896 × 2896 × 1 byte ≈ 8 MB |
| 16 MB | 4096 | 4096 | 4096 × 4096 × 1 byte = 16 MB |
| 32 MB | 5792 | 5792 | 5792 × 5792 × 1 byte ≈ 32 MB |

Or use rectangular buffers:
| Target Size | Rows | Cols | Calculation |
|-------------|------|------|-------------|
| 4 MB | 2048 | 2048 | 4 MB |
| 8 MB | 4096 | 2048 | 8 MB |
| 16 MB | 4096 | 4096 | 16 MB |
| 32 MB | 8192 | 4096 | 32 MB |

#### Step 2: Edit Param.cpp

```cpp
// For 4 MB buffer:
globalBufferType = true;           // Use SRAM (not register file)
globalBufferCoreSizeRow = 2048;
globalBufferCoreSizeCol = 2048;

// For 8 MB buffer:
globalBufferCoreSizeRow = 4096;
globalBufferCoreSizeCol = 2048;

// For 16 MB buffer:
globalBufferCoreSizeRow = 4096;
globalBufferCoreSizeCol = 4096;

// For 32 MB buffer:
globalBufferCoreSizeRow = 8192;
globalBufferCoreSizeCol = 4096;
```

#### Step 3: Recompile

```bash
cd /home/parthpatel/NeuroSim/Inference_pytorch/NeuroSIM
make clean
make
```

#### Step 4: Run Inference

```bash
cd /home/parthpatel/NeuroSim/Inference_pytorch
python3 inference.py --dataset cifar10 --model VGG19 --mode WAGE \
  --inference 1 --subArray 128 --parallelRead 64
```

### Buffer Sweep Script

Create a script to automate buffer sweeps:

```bash
#!/bin/bash
# buffer_sweep.sh

BUFFER_SIZES=("2048 2048" "4096 2048" "4096 4096" "8192 4096")
BUFFER_LABELS=("4MB" "8MB" "16MB" "32MB")

for i in ${!BUFFER_SIZES[@]}; do
    echo "Testing ${BUFFER_LABELS[$i]} buffer..."
    
    # Edit Param.cpp
    ROWS=$(echo ${BUFFER_SIZES[$i]} | cut -d' ' -f1)
    COLS=$(echo ${BUFFER_SIZES[$i]} | cut -d' ' -f2)
    
    # Modify Param.cpp (sed or manual)
    # ... modification code ...
    
    # Recompile
    cd NeuroSIM
    make clean && make
    cd ..
    
    # Run inference
    python3 inference.py --dataset cifar10 --model VGG19 --mode WAGE \
      --inference 1 --subArray 128 --parallelRead 64 \
      > results/buffer_${BUFFER_LABELS[$i]}.log 2>&1
done
```

---

## PE Array Configuration

**PE (Processing Element) array size** determines parallelism.

### Set via Command Line (Recommended)

```bash
# 64×64 PE array
--subArray 64 --parallelRead 64

# 128×128 PE array
--subArray 128 --parallelRead 64

# 256×256 PE array
--subArray 256 --parallelRead 128
```

### Or Edit Param.cpp (Line 203-204)

```cpp
numRowSubArray = 64;    // For 64×64
numColSubArray = 64;

numRowSubArray = 128;   // For 128×128
numColSubArray = 128;

numRowSubArray = 256;   // For 256×256
numColSubArray = 256;
```

**Note:** Command-line parameter `--subArray` overrides Param.cpp values.

### PE Array Sweep (No Recompilation Needed!)

```bash
# Just change --subArray parameter:
for SIZE in 64 128 256; do
    python3 inference.py --dataset cifar10 --model VGG19 --mode WAGE \
      --inference 1 --subArray $SIZE --parallelRead $((SIZE/2)) \
      > results/pe_${SIZE}x${SIZE}.log 2>&1
done
```

---

## Technology Node Selection

**Line 127 in Param.cpp:**

```cpp
technode = 22;  // Technology node in nm
```

### Supported Technology Nodes

| Node (nm) | Type | Features |
|-----------|------|----------|
| 130 | Planar | PTM Model |
| 90 | Planar | PTM Model |
| 65 | Planar | PTM Model |
| 45 | Planar | PTM Model |
| 32 | Planar | PTM Model |
| **22** | **Planar** | **DEFAULT** |
| 14 | FinFET | 4 fins per transistor |
| 10 | FinFET | 3 fins per transistor |
| 7 | FinFET | 2 fins per transistor |
| 5 | FinFET | 2 fins per transistor |
| 3 | FinFET | 2 fins per transistor |
| 2 | Nanosheet (GAA) | 3 nanosheets, backside power |
| 1 | Nanosheet (GAA) | 4 nanosheets, CFET SRAM |

**For my project:** Stick with **22nm** (default) for consistency unless specified otherwise.

---

## Memory Cell Types

**Line 61 in Param.cpp:**

```cpp
memcelltype = 1;  // 1: SRAM, 2: RRAM, 3: FeFET
```

### SRAM (Type 1) - DEFAULT
- **Pros**: Fast, reliable, well-characterized
- **Cons**: Large area, high power in standby
- **Use for**: Your baseline experiments

### RRAM (Type 2)
- **Pros**: High density, non-volatile
- **Cons**: Device variation, slower write
- **Use for**: Comparison studies

### FeFET (Type 3)
- **Pros**: Non-volatile, low voltage
- **Cons**: Limited endurance
- **Use for**: Advanced studies

**For your sweeps:** Use **SRAM (memcelltype = 1)** as baseline.

---

## Quick Config Cheat Sheet

### Your Parameter Sweeps

**Buffer Sweep (requires recompilation for each size):**

| Buffer | Edit Param.cpp | Command |
|--------|----------------|---------|
| 4 MB | `globalBufferCoreSizeRow = 2048;`<br>`globalBufferCoreSizeCol = 2048;` | `--subArray 128` |
| 8 MB | `globalBufferCoreSizeRow = 4096;`<br>`globalBufferCoreSizeCol = 2048;` | `--subArray 128` |
| 16 MB | `globalBufferCoreSizeRow = 4096;`<br>`globalBufferCoreSizeCol = 4096;` | `--subArray 128` |
| 32 MB | `globalBufferCoreSizeRow = 8192;`<br>`globalBufferCoreSizeCol = 4096;` | `--subArray 128` |

**PE Sweep (no recompilation needed):**

| PE Array | Command |
|----------|---------|
| 64×64 | `--subArray 64 --parallelRead 64` |
| 128×128 | `--subArray 128 --parallelRead 64` |
| 256×256 | `--subArray 256 --parallelRead 128` |

### Baseline Configuration

**For all your experiments, use these fixed parameters:**

```bash
--mode WAGE           # 8-bit quantization
--inference 1         # Enable hardware simulation
--cellBit 1           # 1-bit per cell (SRAM)
--ADCprecision 5      # 5-bit ADC
--onoffratio 10       # 10:1 on/off ratio
--batch_size 200      # Standard batch size
```

**In Param.cpp:**
```cpp
memcelltype = 1;      // SRAM
technode = 22;        // 22nm
deviceroadmap = 2;    // LSTP
temp = 300;           // 27°C
```

### Full Sweep Matrix

**3 PE sizes × 4 Buffer sizes × 3 Workloads = 36 runs per simulator**

| Workload | PE | Buffer | Command |
|----------|-----|--------|---------|
| VGG19 | 64×64 | 4 MB | Edit Param.cpp + `--subArray 64` |
| VGG19 | 64×64 | 8 MB | Edit Param.cpp + `--subArray 64` |
| ... | ... | ... | ... |
| ResNet-50 | 256×256 | 32 MB | Edit Param.cpp + `--subArray 256` |

---

## Common Modifications

### Change Operating Temperature

```cpp
temp = 350;  // 77°C (hot environment)
temp = 300;  // 27°C (room temperature) - DEFAULT
temp = 273;  // 0°C (cold environment)
```

### Enable H-Tree Instead of X-Y Bus

```cpp
globalBusType = true;  // Use H-Tree for global interconnect
```

### Use SRAM Buffers Instead of Register Files

```cpp
globalBufferType = true;  // Use SRAM for global buffer
tileBufferType = true;    // Use SRAM for tile buffer
peBufferType = true;      // Use SRAM for PE buffer
```

### Change Clock Frequency

```cpp
clkFreq = 2e9;   // 2 GHz
clkFreq = 1e9;   // 1 GHz - DEFAULT
clkFreq = 500e6; // 500 MHz
```

---

## Verification After Changes

After modifying Param.cpp and recompiling, verify your changes worked:

```bash
# Run a quick test
python3 inference.py --dataset cifar10 --model VGG8 --mode WAGE \
  --inference 1 --subArray 128 --parallelRead 64

# Check the log for buffer/PE size confirmation
grep -i "buffer\|array\|subarray" log/.../log.txt
```

---

## File Locations Reference

| Item | Path |
|------|------|
| Main config | `/home/parthpatel/NeuroSim/Inference_pytorch/NeuroSIM/Param.cpp` |
| Inference script | `/home/parthpatel/NeuroSim/Inference_pytorch/inference.py` |
| Results logs | `/home/parthpatel/NeuroSim/Inference_pytorch/log/default/...` |
| Makefile | `/home/parthpatel/NeuroSim/Inference_pytorch/NeuroSIM/makefile` |
| Your plan | `/home/parthpatel/Project_Edge_and_cloud_systems_for_ML_Algo/parth.md` |

---

**Last Updated:** February 26, 2026  
**Version:** 1.0
