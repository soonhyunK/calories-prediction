# 신체 활동 기반 칼로리 소모량 예측

> 리더보드 RMSE 0.06642 · 2단계 앙상블 파이프라인 · 도메인 지식 기반 피처 엔지니어링

[![Python](https://img.shields.io/badge/Python-3.10+-blue)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-green)](https://xgboost.readthedocs.io/)
[![LightGBM](https://img.shields.io/badge/LightGBM-gray)](https://lightgbm.readthedocs.io/)
[![Result](https://img.shields.io/badge/RMSE-0.06642-brightgreen)]()

---

## 프로젝트 개요

운동 시간, 심박수, 체온, 체중 등 신체 활동 데이터로 칼로리 소모량을 예측하는 회귀 모델입니다. 물리치료사로 쌓은 운동 생리학 지식을 피처 설계에 직접 반영했습니다.

| 항목 | 내용 |
|---|---|
| 과제 유형 | 회귀 (Regression) |
| 데이터셋 | Train 7,500건 / Test 7,500건 |
| 평가 지표 | RMSE |
| 최종 성과 | 리더보드 **RMSE 0.06642** |
| 담당 역할 | 피처 엔지니어링, 2단계 파이프라인 설계, 앙상블 구성 |

---

## 실행 환경 (Requirements)

```
python >= 3.10
```

```bash
pip install -r requirements.txt
```

필요 패키지: `pandas`, `scikit-learn`, `xgboost`, `lightgbm`, `scipy`, `numpy`

---

## 재현성 (Reproducibility)

- **Fold 수**: 9-Fold — 홀수로 설정해 Mode Voting 시 동점 상황 자체를 차단
- **성별 분리 학습**: `is_male` 값에 따라 남/녀 모델을 독립적으로 학습·추론

---

## 사용 기술

`Python` `pandas` `scikit-learn` `XGBoost` `LightGBM` `RandomForest` `scipy`

---

## 핵심 전략

### 1. 2단계 파이프라인
칼로리 값을 한 번에 정밀하게 맞추는 대신, 문제를 두 단계로 쪼갰습니다.
- **Phase 1** — Ridge 회귀(alpha=1e-9)로 소수점 단위의 정밀 예측값을 만듦
- **Phase 2** — RF·XGBoost·LightGBM은 그 소수점을 올림(1)/버림(0)으로만 분류

### 2. 도메인 기반 피처 엔지니어링
운동 생리학 지식을 바탕으로 파생 피처를 설계했습니다.

| 피처명 | 공식 | 설계 근거 |
|---|---|---|
| d_bpm | 운동 시간 × 심박수 | 운동 강도 지표 |
| d_age | 운동 시간 × 나이 | 연령 보정 운동량 |
| d_weight | 운동 시간 × 체중 | 체중 보정 운동량 |
| bpm_age_ratio | 심박수 / 나이 | 신체 효율 지표 |
| bpm_temp | 심박수 × 체온 | 심혈관 부하 지표 |
| dur_sq | 운동 시간² | 비선형 효과 반영 |

### 3. 성별 분리 학습
성별에 따라 운동 반응 패턴이 달라, 하나의 모델로 묶으면 그 차이가 노이즈로 작용했습니다. 남성(`is_male=1`)과 여성(`is_male=0`)을 독립적인 모델로 나눠 학습해 해결했습니다.

### 4. 9-Fold Mode Voting 앙상블
9개 fold에서 나온 예측값을 `scipy.stats.mode()`로 집계해 최종값을 결정했습니다. fold 수를 홀수로 잡아 다수결에서 동점이 나올 수 없게 만들어, 개별 모델의 오류가 상쇄되도록 했습니다.

---

## 모델 비교

| 모델 | 분할 방식 | 선택 이유 |
|---|---|---|
| XGBoost | 깊이 우선(Level-wise) | 높은 정확도, L1/L2 정규화 |
| LightGBM | 리프 우선(Leaf-wise) | 빠른 학습, 높은 메모리 효율 |
| RandomForest | 배깅(Bagging) | 과적합 저항, 앙상블 다양성 확보 |

---

## 주요 인사이트

- Ridge로 소수점까지 복원한 뒤 올림/버림만 분류하는 2단계 구조가 성능의 핵심
- 운동 생리학 지식 기반 피처 설계가 모델 성능 향상에 직접 기여
- 9-Fold를 홀수로 설정해 Mode Voting의 동점 모호성을 원천 차단

---

## 프로젝트 구조

```
calories-prediction/
├── calories_prediction.ipynb    # 전체 실험 코드
├── 칼로리예측_보고서.docx         # 상세 분석 보고서
├── requirements.txt
└── README.md
```

### 실행 방법

```bash
jupyter notebook calories_prediction.ipynb
```

---

## 작성자

권순현 · 오즈코딩스쿨 의료 AI 헬스케어 · 2026
