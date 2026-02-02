# PI Learning - OpenPI VLA Robot Simulation

A benchmark experiment project using Physical Intelligence's **π₀ (pi-zero)** Vision-Language-Action (VLA) model for robot simulation.

## Project Overview

This project evaluates the performance of VLA models in various robot simulation environments using Physical Intelligence's [OpenPI](https://github.com/Physical-Intelligence/openpi) repository.

### What is the π₀ Model?

π₀ is a Vision-Language-Action model developed by Physical Intelligence. It takes images and natural language instructions as input and predicts robot actions. Built on a pre-trained Vision-Language Model (PaliGemma), it uses flow matching to generate continuous actions.

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
| **libero_spatial** | **96.7%** | 29/30 | Spatial relationship understanding (e.g., "move the bowl on the left") |
| **libero_object** | **97.3%** | 109/112 | Object recognition (e.g., "put the tomato sauce in the basket") |
| **libero_goal** | **97.4%** | 419/430 | Goal-oriented tasks |

### ALOHA Sim Benchmark

ALOHA is a dual-arm robot simulation environment.

| Task | Resolution | Description |
|------|------------|-------------|
| Transfer Cube | 640x480 | Transfer a cube from one hand to the other |

## Work Completed

### 1. Environment Setup & Bug Fixes

#### XLA Autotuning Crash Fix
XLA ptxas crashes occurred on RTX 4090. Fixed by adding the following environment variable:

```yaml
# Added to compose.yml
environment:
  - XLA_FLAGS=--xla_gpu_autotune_level=0
```

**Modified files:**
- `examples/libero/compose.yml`
- `examples/aloha_sim/compose.yml`

### 2. Video Quality Improvement (ALOHA Sim)

By default, low-quality videos resized to 224x224 were saved. Modified to save high-quality videos at original resolution (640x480).

**Modified files:**

`examples/aloha_sim/env.py`:
```python
# Store original high-resolution image separately
img_raw_uint8 = image_tools.convert_to_uint8(img_raw)
return {
    "state": gym_obs["agent_pos"],
    "images": {"cam_high": img},
    "images_raw": {"cam_high": img_raw_uint8},  # High-res for video
}
```

`examples/aloha_sim/saver.py`:
```python
# Prioritize high-res image + quality improvement
if "images_raw" in observation:
    im = observation["images_raw"]["cam_high"]
# ...
imageio.mimwrite(out_path, ..., quality=9, output_params=["-crf", "18"])
```

**Result:** Video resolution 224x224 → 640x480, bitrate improved 10x

### 3. Running Benchmarks

```bash
# LIBERO Spatial (default)
xhost +local:docker
SERVER_ARGS="--env LIBERO" docker compose -f examples/libero/compose.yml up

# LIBERO Object
CLIENT_ARGS="--args.task-suite-name libero_object" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# LIBERO Goal
CLIENT_ARGS="--args.task-suite-name libero_goal" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# ALOHA Sim
SERVER_ARGS="--env ALOHA_SIM" docker compose -f examples/aloha_sim/compose.yml up
```

## Saved Results

```
/home/woong/openpi/
├── data/
│   ├── libero/videos/          # LIBERO benchmark videos (44 files)
│   │   ├── rollout_*_success.mp4
│   │   └── rollout_*_failure.mp4
│   └── aloha_sim/videos/       # ALOHA simulation videos
└── ~/.cache/openpi/            # Model checkpoint cache
    ├── pi05_libero/ (11.6GB)   # π₀.5 model for LIBERO
    └── pi0_aloha_sim/ (6.0GB)  # π₀ model for ALOHA
```

## Future Experiment Ideas

### 1. Additional Benchmarks

```bash
# LIBERO 10 - 10 diverse tasks (~45 min)
CLIENT_ARGS="--args.task-suite-name libero_10" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# LIBERO 90 - 90 diverse tasks (~6 hours)
CLIENT_ARGS="--args.task-suite-name libero_90" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up
```

### 2. Other Simulation Environments

| Environment | Description | Requirements |
|-------------|-------------|--------------|
| ALOHA Real | Physical ALOHA robot | Hardware required |
| DROID | General-purpose robot manipulation | Hardware required |
| UR5 | Industrial robot arm | Hardware required |

### 3. Model Fine-tuning

OpenPI supports fine-tuning with custom datasets:

```bash
# Data preparation
XLA_FLAGS=--xla_gpu_autotune_level=0 uv run scripts/compute_norm_stats.py --config-name pi0_aloha_sim

# Run fine-tuning
XLA_FLAGS=--xla_gpu_autotune_level=0 uv run scripts/train.py \
  pi0_aloha_sim \
  --exp-name=my_experiment \
  --overrides training.num_train_steps=30000
```

### 4. Adding New Environments

New simulation environments can be added by following the LeRobot dataset format:

1. Collect data from the environment
2. Convert to LeRobot format
3. Fine-tune the model with the data
4. Evaluate in the new environment

### 5. Prompt Engineering

VLA models accept natural language instructions. Experiment with different prompt styles:

```python
# Basic prompt
"pick up the red cube and place it on the plate"

# Detailed prompt
"carefully grasp the red cube from the table, lift it up, and gently place it in the center of the white plate"

# Step-by-step prompt
"Step 1: Move to the red cube. Step 2: Grasp it. Step 3: Place on plate."
```

### 6. Failure Case Analysis

Analyze the saved failure videos to:
- Identify patterns in failure situations
- Measure failure rates for specific objects/positions
- Distinguish between grasp failures vs placement failures

### 7. Performance Optimization

```bash
# Adjust batch size
SERVER_ARGS="--env LIBERO --batch-size 4"

# Use different precision
SERVER_ARGS="--env LIBERO --precision bf16"
```

## References

- [OpenPI GitHub](https://github.com/Physical-Intelligence/openpi)
- [Physical Intelligence Blog](https://www.physicalintelligence.company/blog/pi0)
- [LIBERO Benchmark](https://github.com/Lifelong-Robot-Learning/LIBERO)
- [ALOHA Project](https://tonyzhaozh.github.io/aloha/)

## Troubleshooting

### XLA ptxas Crash
```
error: ptxas exited with non-zero error code -6
```
**Solution:** Add `XLA_FLAGS=--xla_gpu_autotune_level=0` environment variable

### Wrong Checkpoint Loaded
```
Error: Checkpoint not found for environment
```
**Solution:** Specify `SERVER_ARGS="--env LIBERO"` or `"--env ALOHA_SIM"`

### Docker GPU Access Error
```
Error: could not select device driver
```
**Solution:** Install `nvidia-container-toolkit` and restart Docker

## License

This project is created for educational and research purposes. OpenPI is licensed under Apache 2.0.
