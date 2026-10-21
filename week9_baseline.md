# 9주차 베이스라인 결과

작성 기준일: 2026-10-21

## 구현한 흐름

OSRM 경로 후보를 받아 거리·예상 시간·회전 및 합류 등 기동 유형 수를 비교했습니다.
동일한 출발·도착 구간 안에서 각 지표 순위를 합산해 수치상 우선 후보를 골랐습니다.
환경 정보에 안전 규칙을 적용하고, 근거와 미확인 항목을 LLM 설명에 전달했습니다.

## 실제 OSRM 후보 비교

| 구간 | 경로 ID | 거리(km) | 예상 시간(분) | 기동 유형 수 | 순위 합계 |
|---|---|---:|---:|---:|---:|
| cityhall_to_jamsil | cityhall_to_jamsil_osrm_1 | 14.87 | 16.7 | 5 | 3 |
| cityhall_to_jamsil | cityhall_to_jamsil_osrm_2 | 16.64 | 17.8 | 9 | 6 |
| seoulstation_to_coex | seoulstation_to_coex_osrm_1 | 12.65 | 15.5 | 9 | 3 |
| seoulstation_to_coex | seoulstation_to_coex_osrm_2 | 13.80 | 16.9 | 9 | 5 |

## 안전 규칙 시나리오

- 시나리오 수: 10
- 기대 판정 일치율: 100%
- 판정 분포: blocked: 4, needs_review: 3, ready_for_explanation: 2, no_route: 1
- 입력 데이터 출처: 직접 작성한 구조화 시나리오 (`manual_synthetic`)

## 현재 한계

- 안전 시나리오는 실제 카메라 영상이나 CARLA 데이터가 아니라 수동 작성한 입력입니다.
- OSRM 결과에서 신호·보행자·차선 변경 안전 정보를 얻지 못해 실제 경로는 `needs_review`로 남깁니다.
- 기동 유형 수는 OSRM 경로 안내의 회전·합류 유형을 센 값이며 안전도나 실제 차량 조작 횟수가 아닙니다.
- LLM은 비교 결과를 설명하며 경로 점수나 안전 규칙 판정을 결정하지 않습니다.

## 저장한 산출물

- `data/scenarios/scenarios_10.json`
- `data/scenarios/scenario_evaluation_10.csv`
- `data/scenarios/osrm_route_comparison.csv`
- `data/scenarios/explanation_evidence.json`
- `data/scenarios/integrated_decisions.json`

## OSRM 단계 설명 검증

두 실제 OSRM 경로의 단계 정보를 모두 보존하고, 단계별 LLM 설명을 입력 근거와 대조했습니다.

| 구간 | 설명 단계 수 | 점검 결과 |
|---|---:|---:|
| cityhall_to_jamsil | 9 | 8/8 통과 |
| seoulstation_to_coex | 13 | 9/9 통과 |

- 두 경로 모두 설명에 모든 단계 번호가 순서대로 포함됐습니다.
- 단계별 거리 합계는 OSRM 전체 거리와 일치했습니다.
- 서울역→COEX의 표시된 단계 시간 합계는 15.4분, 전체 시간은 15.5분입니다. 설명은 차이를 표시하고 원인을 단정하지 않았습니다.
- 두 설명 모두 `needs_review`, 미승인 상태, 미확인 항목을 표시했습니다.
- 이 점검은 단계와 설명의 형식·입력 근거 일치를 확인합니다. 실제 주행 안전성을 검증하거나 경로를 승인하지 않습니다.

추가 산출물:
- `data/scenarios/llm_step_explanations.json`
- `data/scenarios/step_explanation_audit.json`
- `data/scenarios/step_explanation_audit_batch.csv`
