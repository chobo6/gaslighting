# G-SAFE
> **Gas Safety Automated Failure Engine**  
> ML 기반 가스 누출 위험률 예측 및 자동차단 Unity 3D 시뮬레이션

---

## 프로젝트 개요

연구소·개발실 환경에서 가스 누출 사고는 초기 대응이 늦을수록 위험도가 급격히 증가하지만, 기존 방식은 단순 농도 임계치 경보에 그쳐 **"언제 차단해야 하는가"** 에 대한 수치 기반 판단 기준이 없습니다.

G-SAFE는 UCI Gas Sensor Array 데이터를 기반으로 **RandomForest / GBR 회귀 모델**이 위험률(risk_score 0~100)을 예측하고, F2-score 기준 최적 threshold를 탐색해 **자동차단 시점을 결정**합니다. 이 과정을 Unity 3D 시뮬레이션으로 시각화하여 비전문가도 직관적으로 이해할 수 있도록 합니다.

> 본 프로젝트는 **교육·연구 목적**의 시뮬레이션입니다.

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| 🎮 **가스 누출 시뮬레이션** | Unity 3D 연구소 씬에서 배관 누출 발생, Particle System으로 가스 확산 시각화 |
| 📡 **센서 데이터 재생** | UCI CSV 시계열 데이터를 1초 단위로 재생, sensor_1~16 실시간 갱신 |
| 🧠 **위험률 예측** | FastAPI를 통해 ML 모델 호출, risk_score 0~100 반환 |
| ⚠️ **Threshold 기반 차단** | F2-score 기준 최적 threshold 탐색, 오탐률 <10% · Recall >0.9 목표 |
| 🔒 **자동차단 처리** | risk_score ≥ threshold 시 밸브 CLOSED, 경보 ON, 환기팬 ON |
| 📊 **실시간 HUD** | 위험률 게이지, 시계열 그래프, 정상/주의/위험/차단 상태 배지 |
| 🗄️ **이벤트 로그** | 차단 이벤트, 센서 로그를 DB에 저장 및 결과 요약 제공 |

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────┐
│  PRESENTATION   Unity 3D 시뮬레이션      │
│  LAYER          Unity HUD · Dashboard   │
├─────────────────────────────────────────┤
│  API LAYER      FastAPI 서버             │
│                 POST /api/predict-risk  │
├──────────────────┬──────────────────────┤
│  ML · DATA       │  데이터 파이프라인    │
│  RF / GBR 모델   │  UCI → 전처리 → 라벨 │
├──────────────────┴──────────────────────┤
│  STORAGE   gas_readings · cutoff_events │
│  LAYER     scenarios · sensor_configs   │
└─────────────────────────────────────────┘
```

### 오프라인 학습 파이프라인
```
UCI CSV → 전처리·정규화 → risk_score 라벨 생성 → RF/GBR 학습 → model.pkl 저장
```

### 실시간 추론 시스템
```
Unity C# → POST JSON → FastAPI predict() → risk_score 반환 → HUD 갱신 → 자동차단
```

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| **Simulation** | Unity 3D, C#, Particle System |
| **Data / ML** | Python, scikit-learn, pandas, numpy, joblib |
| **Backend** | FastAPI, Uvicorn |
| **Database** | SQLite (개발) / PostgreSQL (배포) |
| **Monitoring** | Unity UI, HUD, 시계열 그래프 |

---

## ML 모델 상세

- **데이터셋**: [UCI Gas Sensor Array Drift Dataset](https://archive.ics.uci.edu/ml/datasets/Gas+Sensor+Array+Drift+Dataset) — 16채널 화학 센서, 동적 가스 혼합 시계열
- **라벨 생성**: rule-based — 농도 임계값 기반 점수 + 증가 속도(slope) 가중치 + 지속 시간 패널티
- **모델**: RandomForest Regressor / Gradient Boosting Regressor 비교 후 최적 선택
- **평가 지표**: MAE, RMSE, R² (목표: R² > 0.9)
- **Threshold 탐색**: F2-score 기준으로 후보(60~95) 스캔 → 오탐률 <10%, Recall >0.9 조건 만족 구간에서 선택

---

## DB 스키마

| 테이블 | 설명 |
|--------|------|
| `gas_readings` | 매 타임스텝 센서값 · 위험률 로그 |
| `cutoff_events` | 자동차단 발생 시각 · 위험률 기록 |
| `scenarios` | 시뮬레이션 시나리오 설정값 |
| `sensor_configs` | 센서 채널별 설정 |
| `model_results` | 모델 학습 성능 비교 결과 |

---

## API 명세

### `POST /api/predict-risk`

**Request**
```json
{
  "sensor_1": 0.42,
  "sensor_2": 0.87,
  "...": "...",
  "sensor_16": 0.33
}
```

**Response**
```json
{
  "risk_score": 73.4,
  "risk_level": "위험"
}
```

> 응답 목표: **< 1초** · 실패 시 이전 risk_score 유지 (fallback)

---

## 프로젝트 일정

| 주차 | 기간 | 내용 | 상태 |
|------|------|------|------|
| WEEK 01 | 06.16 – 06.23 | 기획·설계 (요구사항, WBS, ERD, 와이어프레임, 스토리보드) | ✅ 완료 |
| WEEK 02 | 06.23 – 06.30 | 데이터 수집·전처리, Unity 3D 씬·HUD 구현 | 🔄 진행중 |
| WEEK 03 | 06.30 – 07.06 | ML 모델 학습, FastAPI 구현, Unity-API 연동 | 예정 |
| WEEK 04 | 07.06 – 07.13 | 단위·통합 테스트 (TC 15개), 데모 완성, 최종 발표 | 예정 |

---

## 팀 구성

| 이름 | 역할 | 담당 |
|------|------|------|
| **안세웅** | Unity Simulation | 3D 씬 구성, Particle System, 밸브/경보/환기팬 애니메이션 |
| **홍제규** | Backend / API | FastAPI 서버, DB 설계, Unity 연동 API |
| **이성룡** | Test / Demo | TC 15개 수행, 통합 테스트, 데모 시나리오 |
| **장영재** | UI / DB | HUD·그래프 UI, ERD, 와이어프레임, 인프라 |
| **김응석** | Data / ML | UCI 데이터 전처리, RF/GBR 모델 학습, Threshold 탐색 |

---

## 산출물 목록

| 문서 | 파일 |
|------|------|
| 발표 슬라이드 | `presentation_v2/index.html` |
| 요구사항 명세서 | `presentation_v2/requirements.html` |
| WBS | `presentation_v2/wbs.html` |
| 메뉴 트리 / 시나리오 | `presentation_v2/menu_scenario.html` |
| 와이어프레임 | `presentation_v2/wireframe.html` |
| 스토리보드 | `presentation_v2/storyboard.html` |
| 테이블 정의서 / ERD | `presentation_v2/table_definition.html` |
| 테스트 케이스 | `presentation_v2/test_cases.html` |
| 트러블슈팅 / 팀원 회고 | `presentation_v2/troubleshooting.html` |

> 모든 산출물은 HTML 파일로 브라우저에서 바로 열람 가능합니다.

---

## 기대효과

1. **수치 기반 의사결정** — F2-score 최적 threshold로 "언제 차단할지" 객관적 기준 제공
2. **직관적 시각화** — Unity 3D로 비전문가도 누출→위험→차단 흐름을 한눈에 파악
3. **확장 가능한 구조** — CSV 교체만으로 새 시나리오 반영, 모델 교체 없이 threshold 재탐색

---

## 라이선스

본 프로젝트는 교육·연구 목적으로 제작되었습니다.  
UCI Gas Sensor Array 데이터셋 사용 — [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Gas+Sensor+Array+Drift+Dataset)
