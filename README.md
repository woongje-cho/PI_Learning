# PI Learning - OpenPI VLA Robot Simulation

A benchmark experiment project using Physical Intelligence's **π₀ (pi-zero)** Vision-Language-Action (VLA) model for robot simulation.

## Project Overview

This project evaluates the performance of VLA models in various robot simulation environments using Physical Intelligence's [OpenPI](https://github.com/Physical-Intelligence/openpi) repository.

### What is the π₀ Model?

π₀ is a Vision-Language-Action model developed by Physical Intelligence. It takes images and natural language instructions as input and predicts robot actions. Built on a pre-trained Vision-Language Model (PaliGemma), it uses flow matching to generate continuous actions.

## Repository Structure

```
PI_Learning/
├── README.md
├── modified_files/              # Modified source files (copy to openpi)
│   ├── aloha_sim/
│   │   ├── compose.yml          # XLA crash fix
│   │   ├── env.py               # High-res image capture
│   │   └── saver.py             # Video quality improvement
│   └── libero/
│       └── compose.yml          # XLA crash fix
├── patches/                     # Git diff patches
│   ├── aloha_sim.patch
│   └── libero_compose.patch
├── videos/                      # Benchmark result videos
│   └── libero/                  # 44 rollout videos (success/failure)
└── results/
    └── benchmark_summary.md     # Detailed results
```

## Quick Start

### 1. Clone OpenPI
```bash
git clone https://github.com/Physical-Intelligence/openpi.git
cd openpi
git submodule update --init --recursive
```

### 2. Apply Our Modifications
```bash
# Copy modified files (recommended)
cp PI_Learning/modified_files/libero/compose.yml openpi/examples/libero/
cp PI_Learning/modified_files/aloha_sim/* openpi/examples/aloha_sim/

# Or apply patches
cd openpi
git apply ../PI_Learning/patches/libero_compose.patch
git apply ../PI_Learning/patches/aloha_sim.patch
```

### 3. Run Benchmarks
```bash
xhost +local:docker
SERVER_ARGS="--env LIBERO" docker compose -f examples/libero/compose.yml up
```

## Experiment Environment

| Component | Specification |
|-----------|---------------|
| OS | Ubuntu 22.04 |
| GPU | NVIDIA RTX 4090 (24GB VRAM) |
| CUDA | 12.2 |
| Docker | 29.2.0 |
| nvidia-container-toolkit | Installed |

## Benchmark Results

### LIBERO Benchmark

LIBERO is a benchmark containing various manipulation tasks in tabletop environments.

| Task Suite | Success Rate | Episodes | Description |
|------------|--------------|----------|-------------|
| **libero_spatial** | **96.7%** | 29/30 | Spatial relationship understanding |
| **libero_object** | **97.3%** | 109/112 | Object recognition |
| **libero_goal** | **97.4%** | 419/430 | Goal-oriented tasks |

### ALOHA Sim Benchmark

| Task | Resolution | Description |
|------|------------|-------------|
| Transfer Cube | 640x480 | Transfer a cube from one hand to the other |

### Sample Videos

Check the `videos/libero/` folder for 44 rollout videos showing both successful and failed attempts:
- `rollout_*_success.mp4` - Successful task completions
- `rollout_*_failure.mp4` - Failed attempts for analysis

## Modifications Made

### 1. XLA Autotuning Crash Fix

XLA ptxas crashes occurred on RTX 4090. Fixed by adding environment variable:

```yaml
environment:
  - XLA_FLAGS=--xla_gpu_autotune_level=0
```

**Files:** `modified_files/libero/compose.yml`, `modified_files/aloha_sim/compose.yml`

### 2. Video Quality Improvement (ALOHA Sim)

Improved video quality from 224x224 to 640x480 resolution with better encoding.

**Files:** `modified_files/aloha_sim/env.py`, `modified_files/aloha_sim/saver.py`

## Running Different Benchmarks

```bash
# LIBERO Spatial (default)
SERVER_ARGS="--env LIBERO" docker compose -f examples/libero/compose.yml up

# LIBERO Object
CLIENT_ARGS="--args.task-suite-name libero_object" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# LIBERO Goal
CLIENT_ARGS="--args.task-suite-name libero_goal" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# LIBERO 10 (10 diverse tasks, ~45 min)
CLIENT_ARGS="--args.task-suite-name libero_10" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# LIBERO 90 (90 diverse tasks, ~6 hours)
CLIENT_ARGS="--args.task-suite-name libero_90" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# ALOHA Sim
SERVER_ARGS="--env ALOHA_SIM" docker compose -f examples/aloha_sim/compose.yml up
```

## Future Experiment Ideas

### 1. Model Fine-tuning

```bash
XLA_FLAGS=--xla_gpu_autotune_level=0 uv run scripts/train.py \
  pi0_aloha_sim \
  --exp-name=my_experiment \
  --overrides training.num_train_steps=30000
```

### 2. Prompt Engineering

Experiment with different instruction styles:
```python
# Basic
"pick up the red cube and place it on the plate"

# Detailed
"carefully grasp the red cube from the table, lift it up, and gently place it in the center of the white plate"

# Step-by-step
"Step 1: Move to the red cube. Step 2: Grasp it. Step 3: Place on plate."
```

### 3. Failure Case Analysis

Analyze `videos/libero/*_failure.mp4` to identify:
- Grasp failure patterns
- Placement error patterns
- Collision scenarios

### 4. New Environments

Add custom environments using LeRobot dataset format.

## Troubleshooting

| Error | Solution |
|-------|----------|
| `ptxas exited with non-zero error code -6` | Add `XLA_FLAGS=--xla_gpu_autotune_level=0` |
| `Checkpoint not found for environment` | Specify `SERVER_ARGS="--env LIBERO"` |
| `could not select device driver` | Install `nvidia-container-toolkit` |

## References

- [OpenPI GitHub](https://github.com/Physical-Intelligence/openpi)
- [Physical Intelligence Blog](https://www.physicalintelligence.company/blog/pi0)
- [LIBERO Benchmark](https://github.com/Lifelong-Robot-Learning/LIBERO)
- [ALOHA Project](https://tonyzhaozh.github.io/aloha/)

## License

This project is for educational and research purposes. OpenPI is licensed under Apache 2.0.
