# PI Learning - OpenPI VLA Robot Simulation

Physical Intelligence의 **π₀ (pi-zero)** Vision-Language-Action (VLA) 모델을 활용한 로봇 시뮬레이션 벤치마크 실험 프로젝트입니다.

## 프로젝트 개요

이 프로젝트는 Physical Intelligence에서 공개한 [OpenPI](https://github.com/Physical-Intelligence/openpi) 레포지토리를 활용하여 다양한 로봇 시뮬레이션 환경에서 VLA 모델의 성능을 평가합니다.

### π₀ 모델이란?

π₀는 Physical Intelligence에서 개발한 Vision-Language-Action 모델로, 이미지와 자연어 지시를 입력받아 로봇 행동(action)을 예측합니다. 사전 학습된 Vision-Language Model (PaliGemma)을 기반으로 하며, flow matching을 사용하여 연속적인 action을 생성합니다.

## 실험 환경

| 항목 | 스펙 |
|------|------|
| OS | Ubuntu 22.04 |
| GPU | NVIDIA RTX 4090 (24GB VRAM) |
| CUDA | 12.2 |
| Docker | 29.2.0 |
| nvidia-container-toolkit | 설치됨 |

## 벤치마크 결과

### LIBERO Benchmark

LIBERO는 tabletop 환경에서의 다양한 조작 태스크를 포함하는 벤치마크입니다.

| Task Suite | Success Rate | Episodes | 설명 |
|------------|--------------|----------|------|
| **libero_spatial** | **96.7%** | 29/30 | 공간적 관계 이해 (예: "왼쪽 그릇을 옮겨라") |
| **libero_object** | **97.3%** | 109/112 | 객체 인식 (예: "토마토 소스를 바구니에 넣어라") |
| **libero_goal** | **97.4%** | 419/430 | 목표 지향적 태스크 |

### ALOHA Sim Benchmark

ALOHA는 dual-arm 로봇 시뮬레이션 환경입니다.

| Task | 해상도 | 설명 |
|------|--------|------|
| Transfer Cube | 640x480 | 큐브를 한 손에서 다른 손으로 전달 |

## 수행한 작업

### 1. 환경 설정 및 문제 해결

#### XLA Autotuning 크래시 해결
RTX 4090에서 XLA ptxas 크래시가 발생하여 다음 환경변수를 추가:

```yaml
# compose.yml에 추가
environment:
  - XLA_FLAGS=--xla_gpu_autotune_level=0
```

**수정된 파일:**
- `examples/libero/compose.yml`
- `examples/aloha_sim/compose.yml`

### 2. 비디오 품질 개선 (ALOHA Sim)

기본 설정에서는 224x224로 리사이즈된 저화질 비디오가 저장되었습니다. 원본 해상도(640x480)로 고화질 비디오를 저장하도록 수정했습니다.

**수정된 파일:**

`examples/aloha_sim/env.py`:
```python
# 원본 고해상도 이미지를 별도로 저장
img_raw_uint8 = image_tools.convert_to_uint8(img_raw)
return {
    "state": gym_obs["agent_pos"],
    "images": {"cam_high": img},
    "images_raw": {"cam_high": img_raw_uint8},  # 고해상도 비디오용
}
```

`examples/aloha_sim/saver.py`:
```python
# 고해상도 이미지 우선 사용 + 품질 향상
if "images_raw" in observation:
    im = observation["images_raw"]["cam_high"]
# ...
imageio.mimwrite(out_path, ..., quality=9, output_params=["-crf", "18"])
```

**결과:** 비디오 해상도 224x224 → 640x480, 비트레이트 10배 향상

### 3. 벤치마크 실행

```bash
# LIBERO Spatial (기본)
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

## 저장된 결과물

```
/home/woong/openpi/
├── data/
│   ├── libero/videos/          # LIBERO 벤치마크 비디오 (44개)
│   │   ├── rollout_*_success.mp4
│   │   └── rollout_*_failure.mp4
│   └── aloha_sim/videos/       # ALOHA 시뮬레이션 비디오
└── ~/.cache/openpi/            # 모델 체크포인트 캐시
    ├── pi05_libero/ (11.6GB)   # LIBERO용 π₀.5 모델
    └── pi0_aloha_sim/ (6.0GB)  # ALOHA용 π₀ 모델
```

## 향후 실험 아이디어

### 1. 추가 벤치마크 실행

```bash
# LIBERO 10 - 10개의 다양한 태스크 (~45분 소요)
CLIENT_ARGS="--args.task-suite-name libero_10" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up

# LIBERO 90 - 90개의 다양한 태스크 (~6시간 소요)
CLIENT_ARGS="--args.task-suite-name libero_90" SERVER_ARGS="--env LIBERO" \
docker compose -f examples/libero/compose.yml up
```

### 2. 다른 시뮬레이션 환경

| 환경 | 설명 | 필요사항 |
|------|------|----------|
| ALOHA Real | 실제 ALOHA 로봇 | 하드웨어 필요 |
| DROID | 범용 로봇 조작 | 하드웨어 필요 |
| UR5 | 산업용 로봇팔 | 하드웨어 필요 |

### 3. 모델 Fine-tuning

OpenPI는 커스텀 데이터셋으로 fine-tuning을 지원합니다:

```bash
# 데이터 준비
XLA_FLAGS=--xla_gpu_autotune_level=0 uv run scripts/compute_norm_stats.py --config-name pi0_aloha_sim

# Fine-tuning 실행
XLA_FLAGS=--xla_gpu_autotune_level=0 uv run scripts/train.py \
  pi0_aloha_sim \
  --exp-name=my_experiment \
  --overrides training.num_train_steps=30000
```

### 4. 새로운 환경 추가

LeRobot 데이터셋 형식을 따르면 새로운 시뮬레이션 환경을 추가할 수 있습니다:

1. 환경에서 데이터 수집
2. LeRobot 형식으로 변환
3. 데이터로 모델 fine-tuning
4. 새 환경에서 평가

### 5. Prompt Engineering

VLA 모델은 자연어 지시를 받습니다. 다양한 프롬프트 스타일 실험:

```python
# 기본 프롬프트
"pick up the red cube and place it on the plate"

# 더 상세한 프롬프트
"carefully grasp the red cube from the table, lift it up, and gently place it in the center of the white plate"

# 단계별 프롬프트
"Step 1: Move to the red cube. Step 2: Grasp it. Step 3: Place on plate."
```

### 6. 실패 케이스 분석

저장된 failure 비디오들을 분석하여:
- 어떤 상황에서 실패하는지 패턴 파악
- 특정 객체/위치에서의 실패율
- 그립 실패 vs 배치 실패 구분

### 7. 성능 최적화

```bash
# 배치 크기 조정
SERVER_ARGS="--env LIBERO --batch-size 4"

# 다른 precision 사용
SERVER_ARGS="--env LIBERO --precision bf16"
```

## 참고 자료

- [OpenPI GitHub](https://github.com/Physical-Intelligence/openpi)
- [Physical Intelligence 블로그](https://www.physicalintelligence.company/blog/pi0)
- [LIBERO Benchmark](https://github.com/Lifelong-Robot-Learning/LIBERO)
- [ALOHA Project](https://tonyzhaozh.github.io/aloha/)

## 트러블슈팅

### XLA ptxas 크래시
```
error: ptxas exited with non-zero error code -6
```
**해결:** `XLA_FLAGS=--xla_gpu_autotune_level=0` 환경변수 추가

### 잘못된 체크포인트 로드
```
Error: Checkpoint not found for environment
```
**해결:** `SERVER_ARGS="--env LIBERO"` 또는 `"--env ALOHA_SIM"` 명시

### Docker GPU 접근 오류
```
Error: could not select device driver
```
**해결:** `nvidia-container-toolkit` 설치 및 Docker 재시작

## 라이선스

이 프로젝트는 교육 및 연구 목적으로 작성되었습니다. OpenPI는 Apache 2.0 라이선스를 따릅니다.
