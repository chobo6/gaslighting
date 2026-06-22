# G-SAFE
> **Gas Safety Automated Failure Engine**
> 가스 누출 위험 감지부터 자동차단·자동복구까지, 사람 개입 없는 **자동화 폐루프 제어** Unity 3D 시뮬레이션

---

## 프로젝트 개요

연구소·개발실 환경에서 가스 누출 사고는 초기 대응이 늦을수록 위험도가 급격히 증가하지만, 기존 방식은 단순 농도 임계치 경보에 그쳐 **"누가 언제 차단할 것인가"** 를 사람에게 맡깁니다.

G-SAFE는 가스 누출을 **감지 → 판단 → 자동차단 → 자동복구** 까지 사람 개입 없이 처리하는 **자동화 폐루프(closed-loop) 제어 시스템**입니다. 농도·증가율·지속시간 기반 규칙으로 위험률을 산출하고, 상태머신이 자동으로 등급을 전이시키며, 차단 시퀀스(밸브→경보→환기팬)를 순차 실행하고, 안전값 복귀 시 인터락 조건을 검증해 자동으로 정상 복구합니다. 이 전 과정을 Unity 3D로 시각화해 비전문가도 직관적으로 이해할 수 있도록 합니다.

> 본 프로젝트는 **교육·연구 목적**의 시뮬레이션입니다.

### 설계 방향 (2026-06-22 전환)

가스 농도→위험은 LEL 등 물리·법규 상수로 정해지는 **결정론적 문제**입니다. 규칙으로 정답(라벨)을 만들고 ML로 그 규칙을 다시 학습하면 순환학습(rule-in/rule-out)이 되어 ML의 실질 가치가 낮습니다. 따라서 G-SAFE는 **데이터 분석 중심에서 자동화 중심으로 초점을 전환**했습니다.

- **ML은 위험률을 내려주는 얇은 API로 축소** (RandomForest 1개, 비교·튜닝·EDA 제거)
- **Unity 자동화(감지→판단→차단→복구 폐루프)를 프로젝트의 핵심 가치로 확장**
- 평가 지표를 R²·MAE → **자동화 지표**(차단 지연, 시퀀스 완결성, 무인 완주, 자동 복구)로 교체

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| 🎮 **가스 누출 시뮬레이션** | Unity 3D 연구소 씬에서 배관 누출 발생, Particle System으로 가스 확산 시각화 |
| 📡 **센서 데이터 자동 재생** | UCI CSV 시계열 데이터를 1초 단위로 자동 재생, sensor_1~16 실시간 갱신 |
| 🧠 **위험률 산출 API** | FastAPI로 위험률(risk_score 0~100) 반환 (RF 모델 + 규칙 로직) |
| 🔁 **상태머신 자동 전이** | 정상→주의→위험→차단→복구를 사람 개입 없이 자동 전이 |
| ⚙️ **순차 차단 시퀀스** | risk ≥ threshold 시 밸브 닫힘 → 경보 ON → 환기팬 ON 순차 자동 실행 |
| 🔓 **자동 복구 / 인터락** | 환기 후 농도 안전값 복귀 시 자동 밸브 재개방·경보 해제 (안전 조건 미충족 시 차단) |
| 📊 **의사결정 근거 HUD** | "왜 지금 차단했는가" 실시간 표시 + 위험률 게이지·시계열 그래프 |
| 🗄️ **자동 이벤트 로그** | 차단 이벤트·센서 로그를 DB에 자동 저장 및 결과 요약 |
| 🤖 **무인 자동 데모 모드** | 시작 1회 클릭으로 정상→누출→차단→복구 전 과정 무인 완주 |

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────────────┐
│  PRESENTATION   Unity 3D 시뮬레이션 (자동화 핵심) │
│  LAYER          StateMachine · CutoffSequence    │
│                 AutoRecovery · DecisionHUD       │
├─────────────────────────────────────────────────┤
│  API LAYER      FastAPI 서버                      │
│                 POST /api/predict-risk           │
├──────────────────┬──────────────────────────────┤
│  ML · 규칙        │  데이터 파이프라인            │
│  RF 모델(얇은 API)│  UCI → 전처리 → 규칙 라벨    │
├──────────────────┴──────────────────────────────┤
│  STORAGE   gas_readings · cutoff_events          │
│  LAYER     scenarios · sensor_configs            │
└─────────────────────────────────────────────────┘
```

### 자동화 폐루프 (핵심 흐름)
```
SensorPlayer(자동 재생) → ApiClient(자동 호출) → StateMachine(자동 판단)
   → CutoffSequence(밸브→경보→환기 순차) → DB 자동 기록
   → AutoRecovery(안전 복귀 시 자동 재개방) → [다음 시나리오 자동 전환]
                         ※ 전 과정 사람 개입 0
```

### 오프라인 준비 (1회)
```
UCI CSV → 전처리·정규화 → 규칙 기반 risk_score 라벨 → RF 학습 → model.pkl 저장
```

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| **Simulation / 자동화** | Unity 3D, C#, 상태머신, Particle System |
| **Data / ML (축소)** | Python, scikit-learn(RandomForest), pandas, numpy, joblib |
| **Backend** | FastAPI, Uvicorn |
| **Database** | SQLite (개발) / PostgreSQL (배포) |
| **Monitoring** | Unity UI, 의사결정 근거 HUD, 시계열 그래프 |

---

## ML / 규칙 상세 (축소판)

- **데이터셋**: [UCI Gas Sensor Array Drift Dataset](https://archive.ics.uci.edu/ml/datasets/Gas+Sensor+Array+Drift+Dataset) — 16채널 화학 센서, 동적 가스 혼합 시계열
- **규칙(핵심 판단 로직)**: 농도 임계값(LEL 25%) 기준 점수 + 증가 속도(slope) 가중치 + 지속 시간 패널티 → `risk_score` 0~100. 이 규칙은 Unity 상태머신의 판단 기준으로도 재사용
- **모델**: RandomForest Regressor **1개**, 기본 파라미터 고정 (RF/GBR 비교·GridSearchCV·EDA 리포트 제거)
- **Threshold**: 탐색 대신 **고정값 1개**(예: 70) — "LEL 기준 위험 등급 시작 지점"으로 근거 명시
- **역할**: ML은 위험률을 내려주는 **얇은 API**일 뿐, 프로젝트의 무게중심은 Unity 자동화에 있음

---

## DB 스키마

| 테이블 | 설명 |
|--------|------|
| `gas_readings` | 매 타임스텝 센서값 · 위험률 로그 |
| `cutoff_events` | 자동차단 발생 시각 · 위험률 · 복구 시각 기록 |
| `scenarios` | 시뮬레이션 시나리오 설정값 (연구소/개발실/배관실) |
| `sensor_configs` | 센서 채널별 설정 |
| `automation_metrics` | 차단 지연·시퀀스 완결성·복구 성공률 등 자동화 지표 기록 |

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
  "risk_level": "위험",
  "response_ms": 42
}
```

> 응답 목표: **< 1초** · 실패 시 이전 risk_score 유지 (fallback) · 3회 연속 실패 시 로컬 데모 모드 자동 전환

---

## 자동화 지표 (평가 기준)

| 지표 | 목표 |
|------|------|
| **차단 지연시간** (위험 감지 → 밸브 닫힘) | < 1초 |
| **시퀀스 완결성** (밸브·경보·환기 모두 실행) | 100% |
| **자동 복구 성공률** (안전 복귀 시 무인 재개방) | 측정·기록 |
| **무인 데모 완주** (사람 개입 횟수) | 0회 |
| **렌더링 성능** | 30fps+ |

---

## 프로젝트 일정

| 주차 | 기간 | 내용 | 상태 |
|------|------|------|------|
| WEEK 01 | 06.16 – 06.23 | 기획·설계 (요구사항, WBS, ERD, 와이어프레임, 스토리보드) | ✅ 완료 |
| WEEK 02 | 06.23 – 06.30 | ML 종료(model.pkl) + Unity 씬·HUD 골격 + **API↔Unity 연동** | 🔄 진행중 |
| WEEK 03 | 06.30 – 07.06 | **자동화 핵심** (상태머신·차단 시퀀스·근거 HUD·무인 데모·자동 복구) | 예정 |
| WEEK 04 | 07.06 – 07.13 | 테스트(TC) + 자동화 지표 측정 + 무인 데모 완주 + 최종 발표 | 예정 |

---

## 팀 구성

| 이름 | 역할 | 담당 |
|------|------|------|
| **안세웅** | Unity Simulation / 자동화 | 3D 씬, Particle, 상태머신, 차단 시퀀스, 무인 데모, 자동 복구 |
| **홍제규** | Backend / API | FastAPI 서버, DB 설계, Unity 연동 API |
| **이성룡** | Test / Demo | TC 수행, 통합 테스트, 자동화 지표 측정, 데모 시나리오 |
| **장영재** | UI / DB / 자동화 | 의사결정 근거 HUD·그래프, 시나리오 전환, ERD, 와이어프레임 |
| **김응석** | Data / 규칙 | UCI 전처리, RF 학습(1회), **규칙 로직 → 상태머신 판단 기준 지원** |

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
> ⚠️ 방향 전환에 따라 일부 HTML 산출물(RTM·WBS·스토리보드·테스트 케이스)은 WEEK 02 중 갱신 예정.

---

## 기대효과

1. **무인 안전 대응** — 감지부터 복구까지 사람 개입 없이 자동 처리, 초기 대응 지연 제거
2. **직관적 시각화** — Unity 3D로 비전문가도 누출→판단→차단→복구 흐름을 한눈에 파악
3. **투명한 의사결정** — "왜 지금 차단했는가"를 HUD로 실시간 제시
4. **확장 가능한 구조** — CSV·시나리오 교체만으로 새 환경 반영, 규칙 기반이라 재현·검증 용이

---

## 라이선스

본 프로젝트는 교육·연구 목적으로 제작되었습니다.
UCI Gas Sensor Array 데이터셋 사용 — [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Gas+Sensor+Array+Drift+Dataset)
