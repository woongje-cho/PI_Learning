# Benchmark Results Summary

Experiment Date: 2025-02-01 ~ 2025-02-02

## System Information
- GPU: NVIDIA RTX 4090 (24GB)
- CUDA: 12.2
- OS: Ubuntu 22.04
- Docker: 29.2.0

## LIBERO Benchmark Results

### libero_spatial
- **Success Rate: 96.7% (29/30)**
- Example task: "pick up the black bowl on the cookie box and place it on the plate"
- Tests spatial relationship understanding

### libero_object
- **Success Rate: 97.3% (109/112)**
- Example task: "pick up the salad dressing and place it in the basket"
- Tests object recognition and manipulation

### libero_goal
- **Success Rate: 97.4% (419/430)**
- Example task: "put the bowl on the plate"
- Tests goal-oriented task execution

## Checkpoints Used

| Checkpoint | Size | Purpose |
|------------|------|---------|
| pi05_libero | 11.6GB | LIBERO benchmark |
| pi0_aloha_sim | 6.0GB | ALOHA simulation |

## Video Results

Total of 44 video files generated:
- Success cases: 22
- Failure cases: 22

Failure case analysis points:
1. Grasp failure (failed to grasp the object)
2. Placement error (placed in wrong position)
3. Collision (collided with other objects)

## Execution Time

| Benchmark | Episodes | Estimated Time |
|-----------|----------|----------------|
| libero_spatial | 30 | ~5 min |
| libero_object | 500 | ~40 min |
| libero_goal | 500 | ~40 min |
| libero_10 | 500 | ~45 min |
| libero_90 | 4500 | ~6 hours |
