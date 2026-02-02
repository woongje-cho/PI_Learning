# Benchmark Results Summary

실험 일자: 2025-02-01 ~ 2025-02-02

## 시스템 정보
- GPU: NVIDIA RTX 4090 (24GB)
- CUDA: 12.2
- OS: Ubuntu 22.04
- Docker: 29.2.0

## LIBERO Benchmark Results

### libero_spatial
- **Success Rate: 96.7% (29/30)**
- 태스크 예시: "pick up the black bowl on the cookie box and place it on the plate"
- 공간적 관계 이해 능력 테스트

### libero_object
- **Success Rate: 97.3% (109/112)**
- 태스크 예시: "pick up the salad dressing and place it in the basket"
- 객체 인식 및 조작 능력 테스트

### libero_goal
- **Success Rate: 97.4% (419/430)**
- 태스크 예시: "put the bowl on the plate"
- 목표 지향적 태스크 수행 능력 테스트

## 사용된 체크포인트

| 체크포인트 | 크기 | 용도 |
|-----------|------|------|
| pi05_libero | 11.6GB | LIBERO 벤치마크 |
| pi0_aloha_sim | 6.0GB | ALOHA 시뮬레이션 |

## 비디오 결과물

총 44개의 비디오 파일 생성:
- 성공 케이스: 22개
- 실패 케이스: 22개

실패 케이스 분석 포인트:
1. 그립 실패 (객체를 잡지 못함)
2. 배치 오류 (잘못된 위치에 놓음)
3. 충돌 (다른 객체와 충돌)

## 실행 시간

| 벤치마크 | 에피소드 수 | 예상 시간 |
|----------|------------|----------|
| libero_spatial | 30 | ~5분 |
| libero_object | 500 | ~40분 |
| libero_goal | 500 | ~40분 |
| libero_10 | 500 | ~45분 |
| libero_90 | 4500 | ~6시간 |
