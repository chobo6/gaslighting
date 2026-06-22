# G-SAFE 프로젝트 TODO

> 기준일: 2026-06-22 | 전체 일정: 2026.06.16 – 07.13 (4주)
> **프로젝트 초점: 데이터 분석 → 자동화 (Unity 폐루프 제어) 중심으로 전환**
> 목표 지표: 차단 지연 < 1s · 시퀀스 완결성 100% · 무인 데모 완주 · 자동 복구 성공 · 30fps+

---

## 방향 전환 요약 (2026-06-22)

가스 농도→위험은 LEL 등 물리·법규 상수로 정해지는 **결정론적 문제**입니다.
규칙으로 라벨을 만들고 ML로 그 규칙을 재학습하는 기존 구조는 순환학습(rule-in/rule-out)이라
ML의 실질 가치가 낮습니다. 따라서:

- **ML은 "1주짜리 얇은 위험률 API"로 축소** (RF 1개만, 비교·튜닝·EDA 삭제)
- **Unity 자동화(감지→판단→차단→복구 폐루프)를 프로젝트의 주인공으로 확장**
- 평가 지표를 R²·MAE → **자동화 지표**(차단 지연, 시퀀스 완결성, 무인 완주, 자동 복구)로 교체

---

## 기획 단계 (WEEK 01) ✅ 완료

| 산출물 | 파일 | 비고 |
|--------|------|------|
| 요구사항 명세서 | `presentation_v2/requirements.html` | 기능 17 + 비기능 8 + RTM 탭 |
| WBS | `presentation_v2/wbs.html` | 36 tasks, 4주 Gantt |
| 메뉴 트리 & 시나리오 | `presentation_v2/menu_scenario.html` | 4단계 메뉴 + 4 시나리오 |
| 와이어프레임 | `presentation_v2/wireframe.html` | 시뮬레이션 + 결과 상세 2화면 |
| 스토리보드 | `presentation_v2/storyboard.html` | 5 Scene |
| 테이블 정의서 / ERD | `presentation_v2/table_definition.html` | 5개 테이블 |
| 테스트 케이스 | `presentation_v2/test_cases.html` | 15개 TC |
| 트러블슈팅 & 회고 초안 | `presentation_v2/troubleshooting.html` | 예상 이슈 4건 |
| 발표 슬라이드 | `presentation_v2/index.html` | 9슬라이드 |
| API 명세서 | `presentation_v2/api_spec.html` | 5 엔드포인트, 에러 형식 |
| RTM | `presentation_v2/requirements.html` RTM 탭 | 커버리지 84% (21/25) |
| README | `README.md` | 프로젝트 개요 / 아키텍처 / 팀 |
| GitHub 연결 & push | `https://github.com/chobo6/gaslighting.git` | main 브랜치 |

> ⚠️ **기획 문서 갱신 필요**: 방향 전환에 따라 아래 HTML 산출물도 추후 업데이트 대상
> (requirements RTM, wbs, storyboard, test_cases). WEEK 02 착수 후 반영.

### 기획 잔여 (선택사항)

- [ ] **와이어프레임 화면 보완** (`presentation_v2/wireframe.html` 탭 추가)
  - [ ] 대시보드 화면 — 시스템 상태 요약 / 최근 이력 / 자동화 지표 카드
  - [ ] 시나리오 선택 화면 — 연구소 / 개발실 / 배관실 카드 + 설정값

---

## 개발 단계 — WEEK 02 (06.23 – 06.30)

> **목표: ML 완전 종료(model.pkl 완성) + Unity 씬·HUD 골격 + API↔Unity 연동 1회 성공**
> (연동을 WEEK 03 → WEEK 02로 앞당겨 WEEK 03을 자동화 로직에만 집중)

### 데이터 / ML (축소) — 담당: 김응석 *(완료 후 Unity 상태머신 규칙 지원으로 이동)*

- [ ] **UCI 데이터셋 다운로드 및 검증**
  - [ ] [UCI Gas Sensor Array Drift Dataset](https://archive.ics.uci.edu/ml/datasets/Gas+Sensor+Array+Drift+Dataset) 다운로드
  - [ ] 컬럼 확인: `time`, `sensor_1~16`, `gas_concentration` 누락 없음 → TC-DATA-001
  - [ ] (간소화) 전체 행 수 · 결측값만 확인. ~~상세 EDA 리포트 작성~~ **삭제**

- [ ] **전처리 스크립트** (`data/preprocess.py`)
  - [ ] 결측값 선형 보간 (또는 행 제거 기준 명시)
  - [ ] `MinMaxScaler`로 `sensor_1~16` 정규화, `scaler.pkl` 저장
  - [ ] 전처리 결과 `data/processed.csv` 저장

- [ ] **라벨/규칙 생성 함수** (`data/label_generator.py`) ⭐ *사실상 핵심 판단 로직 — 자동화에 재사용*
  - [ ] 1단계: `gas_concentration` 임계값(예: LEL 25%) 기준 기본 점수 산출
  - [ ] 2단계: 1초 slope(`np.gradient`) 가중치 부여 (급등 시 +보정)
  - [ ] 3단계: 위험 지속 시간(연속 초과 카운트) 패널티 적용
  - [ ] 최종 `risk_score` 0~100 클리핑 → TC-DATA-002
  - [ ] **이 규칙 로직은 Unity StateMachine의 판단 기준으로 그대로 재사용**

- [ ] **RandomForest 학습 (1회로 종료)** (`model/train_rf.py`)
  - [ ] `n_estimators=200, max_depth=10, random_state=42` **기본값 고정** (튜닝 없음)
  - [ ] `model/best_model.pkl` 저장 (joblib)
  - [ ] ~~GBR 학습 / RF vs GBR 비교 / GridSearchCV~~ **전부 삭제**

- [ ] **Threshold 고정** (`model/threshold_config.py`)
  - [ ] ~~60~95 전체 스캔~~ → **고정값 1개 선정** (예: 70)
  - [ ] 근거 한 줄 명시: "LEL 기준 위험 등급 시작 지점" → TC-ML-002 대체

### Unity / 시뮬레이션 — 담당: 안세웅

- [ ] **Unity 프로젝트 초기 설정**
  - [ ] Unity 6000.x 프로젝트 생성 (3D URP 템플릿)
  - [ ] `.gitignore` Unity용 설정 (Library/, Temp/, Build/ 제외)
  - [ ] GitHub 레포에 `unity/` 폴더로 push

- [ ] **연구소 3D 씬 구성** (`Assets/Scenes/Lab.unity`)
  - [ ] 바닥 · 벽 · 천장 Plane/Cube 배치
  - [ ] 배관 (Cylinder) + 밸브 오브젝트 배치
  - [ ] 누출 지점 `Empty (LeakPoint)` 생성 — Particle System 부모로 사용
  - [ ] 카메라 위치 고정 (발표용 최적 앵글 1개)

- [ ] **Particle System — 가스 확산** (`Assets/Prefabs/GasLeak.prefab`)
  - [ ] `Emission Rate`: 0 (기본) → 50 (누출 시) 전환 스크립트
  - [ ] 색상: 반투명 초록(정상) → 노랑 → 빨강(위험) 단계별 Color over Lifetime
  - [ ] `Max Particles` 200 제한 (성능 보장 · REQ-NF 30fps+)

- [ ] **상태 오브젝트 애니메이션** (`Assets/Scripts/SimObjects.cs`)
  - [ ] 밸브: `localRotation.z` 0→90° (OPEN→CLOSED) `Lerp` 0.5s
  - [ ] 경보등: `PointLight.intensity` 깜빡임 코루틴 (0.5s 주기)
  - [ ] 환기팬: `localRotation.y` 지속 회전 (`rpm` 파라미터로 제어)

### HUD / UI — 담당: 장영재

- [ ] **Canvas 구성** (`Assets/UI/HUD.canvas`, Screen Space - Overlay)
  - [ ] 위험률 게이지 바: `Image.fillAmount` = `risk_score / 100`
  - [ ] threshold 기준선: 게이지 바 위 `RectTransform.anchoredPositionY` 고정
  - [ ] 위험 등급 배지 `Text`: 정상(#00ff88) / 주의(#ffaa00) / 위험(#ff3333) / 차단(#888888)
  - [ ] 밸브 · 경보 · 환기 상태 아이콘 3개 (Sprite 교체 방식)

- [ ] **CSV 센서 자동 재생** (`Assets/Scripts/SensorPlayer.cs`)
  - [ ] `Resources.Load<TextAsset>` 또는 `StreamingAssets` 경로로 CSV 로드
  - [ ] `InvokeRepeating(1.0f)` 1초 주기 타임스텝 진행
  - [ ] 각 행의 `sensor_1~16` 파싱 → `float[]` 로 `ApiClient.cs`에 전달

### API↔Unity 연동 (WEEK 02로 앞당김) — 담당: 홍제규 + 안세웅

- [ ] **FastAPI 최소 서버 먼저 기동** (`backend/main.py` + `routers/predict.py`)
  - [ ] `POST /api/predict-risk` 1개만 우선 동작 (model.pkl 로드 → predict)
  - [ ] `uvicorn main:app --reload` → `/docs`에서 응답 확인
- [ ] **`ApiClient.cs` 1차** (`Assets/Scripts/ApiClient.cs`)
  - [ ] `UnityWebRequest.Post(url, jsonBody)` 비동기 코루틴
  - [ ] `SensorPlayer.cs`에서 1초마다 `StartCoroutine(RequestRisk(...))` 호출
  - [ ] JSON 응답 파싱 → risk_score 수신 확인 (HUD 게이지 반영)
  - [ ] **WEEK 02 종료 조건: Unity Play → API 호출 → 게이지 움직임 1회 성공**

---

## 개발 단계 — WEEK 03 (06.30 – 07.06) ⭐ 자동화 핵심 주간

> **목표: 감지→판단→차단→복구 폐루프를 사람 개입 0으로 완주**

### 자동화 핵심 — 🔴 필수 (담당 명시)

- [ ] **상태머신 자동 전이** (`Assets/Scripts/StateMachine.cs`) — 담당: 안세웅 + 김응석(규칙)
  - [ ] 상태 정의: 정상(<40) → 주의(40~59) → 위험(60~69) → 차단(≥70, threshold) → 복구
  - [ ] risk_score 입력마다 자동 등급 판정 + 상태 전이 이벤트 발행
  - [ ] 위험 단계 진입 시 차단 카운트다운 트리거
  - [ ] `label_generator.py`의 규칙(농도+slope+지속)을 판단 근거로 동기화

- [ ] **순차 차단 시퀀스** (`Assets/Scripts/CutoffSequence.cs`) — 담당: 안세웅
  - [ ] 차단 트리거 시: 밸브 닫힘(0.5s) → 경보 ON → 환기팬 ON **순차 실행**
  - [ ] 각 단계 타이밍이 눈에 보이게 연출 (단계별 0.3~0.5s 간격)
  - [ ] 시퀀스 완료 플래그 발행 (완결성 검증용) → TC-UNITY-002

- [ ] **의사결정 근거 HUD** (`Assets/Scripts/DecisionHUD.cs`) — 담당: 장영재
  - [ ] "왜 지금 차단했는가" 실시간 표시: "농도 82% · 증가율 급등 → 자동차단"
  - [ ] 현재 상태/threshold/차단까지 남은 예상시간 표시
  - [ ] 위험률 시계열 그래프 (최근 60초 `Queue<float>`, LineRenderer/RawImage)

- [ ] **무인 자동 데모 모드** (`Assets/Scripts/AutoDemo.cs`) — 담당: 안세웅
  - [ ] 시작 버튼 1회 → 정상→누출→차단(→복구) 전 과정 자동 진행
  - [ ] 사람 클릭 0회로 완주 → TC-PRES-001
  - [ ] 발표용 카메라 앵글/속도 고정

### 자동화 확장 — 🟡 선택 (필수 완료 후 진행, 이번 진행 확정)

- [ ] **자동 복구 / 인터락** (`Assets/Scripts/AutoRecovery.cs`) — 담당: 안세웅 + 김응석
  - [ ] 환기 가동 후 농도가 안전값(예: <30) 복귀 감지
  - [ ] 인터락 조건 충족 시 **자동 밸브 재개방 + 경보 해제 + 환기 정지**
  - [ ] 안전 조건 미충족 시 재개방 차단 (인터락 로직) → 자동 복구 성공률 측정

- [ ] **다중 시나리오 자동 전환** (`Assets/Scripts/ScenarioManager.cs`) — 담당: 장영재
  - [ ] 연구소 / 개발실 / 배관실 시나리오 CSV·설정 분리
  - [ ] 데모 모드에서 시나리오 순차 자동 전환

### FastAPI 서버 보강 — 담당: 홍제규

- [ ] **라우터 확장** (`backend/routers/`)
  - [ ] `predict.py` — `risk_level` 변환, `response_ms` 포함 (TC-API-001)
  - [ ] `logs.py` — `POST /api/logs/cutoff` (`cutoff_events` INSERT, TC-DB-002)
  - [ ] `meta.py` — `GET /api/health`, `/scenarios`, `/model-performance`
- [ ] **입력 검증** — Pydantic `Field(ge=0.0, le=1.0)`, 누락 시 HTTP 422 (TC-API-002)
- [ ] **DB 초기화** (`db/init_db.py`)
  - [ ] `gas_readings`, `cutoff_events`, `scenarios`, `sensor_configs`, `automation_metrics` 5개 테이블
  - [ ] `scenarios` 초기 데이터 INSERT (연구소/개발실/배관실 3건)
- [ ] **ApiClient.cs 강화**
  - [ ] 타임아웃 2초, 실패 시 이전 risk_score 유지 (fallback · REQ-NF-006)
  - [ ] 3회 연속 실패 시 로컬 데모 모드 자동 전환 (API 의존성 제거)
  - [ ] 차단 발생 시 `POST /api/logs/cutoff` 자동 호출

---

## 테스트 단계 — WEEK 04 (07.06 – 07.13)

### 단위 / 통합 테스트 — 담당: 이성룡

| TC ID | 항목 | 수행일 | 담당 |
|-------|------|--------|------|
| TC-DATA-001 | UCI 데이터 필수 컬럼 확인 | 07-06 | 김응석 |
| TC-DATA-002 | risk_score 0~100 범위 검증 | 07-06 | 김응석 |
| TC-ML-001 | RF 학습 성공 + model.pkl 생성 | 07-06 | 김응석 |
| TC-ML-002 | Threshold 고정값 선정 근거 확인 | 07-06 | 김응석 |
| TC-API-001 | POST /predict-risk 정상 응답 (<1s) | 07-07 | 홍제규 |
| TC-API-002 | 잘못된 입력 → HTTP 422 반환 | 07-07 | 홍제규 |
| TC-DB-001 | 시뮬레이션 후 gas_readings 저장 | 07-07 | 장영재 |
| TC-DB-002 | threshold 초과 → cutoff_events 저장 | 07-07 | 장영재 |
| TC-UNITY-001 | Particle System 누출 지점 활성화 | 07-08 | 안세웅 |
| **TC-UNITY-002** | **차단 시퀀스 완결성** (밸브·경보·환기 3개 모두 실행) | 07-08 | 안세웅 |
| TC-UNITY-003 | HUD 위험률 1초 이하 주기 갱신 | 07-08 | 장영재 |
| **TC-AUTO-001** | **차단 지연시간 측정** (위험 감지→밸브 닫힘 < 1s) | 07-09 | 이성룡 |
| **TC-AUTO-002** | **자동 복구 / 인터락** (안전 복귀 시 자동 재개방) | 07-09 | 안세웅 |

### 통합 테스트 — 담당: 이성룡

| TC ID | 항목 | 수행일 |
|-------|------|--------|
| TC-INT-001 | Unity → API → HUD 실시간 반영 | 07-10 |
| TC-INT-002 | 정상 시나리오 (risk < threshold) — 차단 미발생 | 07-10 |
| TC-INT-003 | 위험 시나리오 — 자동차단 + 시퀀스 + DB 저장 | 07-10 |
| TC-INT-004 | 다중 시나리오 자동 전환 정상 동작 | 07-10 |
| TC-PRES-001 | **무인 자동 데모** 사람 개입 0회 완주 | 07-13 |

### 발표 준비 — 담당: 전체

- [ ] 자동화 지표 측정·정리 (차단 지연 / 시퀀스 완결성 / 무인 완주 / 자동 복구 성공률)
- [ ] 트러블슈팅 문서 실제 이슈로 업데이트 (`presentation_v2/troubleshooting.html`)
- [ ] 팀원 회고 최종 작성
- [ ] 발표 슬라이드 — 방향 전환(분석→자동화) 스토리 반영
- [ ] 발표 환경 사전 테스트 — 30fps+ 유지, 해상도 1920×1080 확인
- [ ] 최종 GitHub push + 버전 태그 (`git tag v1.0.0`)

---

## 자동화 흐름 (전체 그림)

```
[무인 자동 데모 모드 시작]
        ↓
SensorPlayer → CSV 1초씩 재생 (정상 → 누출 구간)
        ↓
ApiClient → FastAPI predict() 자동 호출 → risk_score 수신
        ↓
StateMachine → 등급 자동 판단 + 상태 전이
        ├ < 40   정상  : 초록 게이지
        ├ 40~59  주의  : 노랑 + HUD 경고
        ├ 60~69  위험  : 빨강 + 차단 카운트다운
        └ ≥ 70   차단  : ▼ 차단 시퀀스 트리거
                              ↓
        CutoffSequence → 밸브 닫힘(0.5s) → 경보 ON → 환기팬 ON (순차)
                              ↓
        DecisionHUD → "농도 82% · 증가율 급등 → 자동차단" 근거 표시
                              ↓
        cutoff_events DB 자동 기록
                              ↓
        AutoRecovery → 농도 안전값 복귀 감지 → 자동 재개방 + 경보 해제
                              ↓
        [다중 시나리오면] ScenarioManager → 다음 시나리오 자동 전환
```

---

## RTM 미커버 요구사항 처리 계획

| 요구사항 | 이유 | 처리 방법 |
|----------|------|-----------|
| REQ-F-011 위험 원인 설명 | TC 없음 | `DecisionHUD.cs` 의사결정 근거 표시로 커버 |
| REQ-F-013 수동 파라미터 조정 | TC 없음 | WEEK 04 슬라이더 UI 구현 후 수동 확인 |
| REQ-NF-005 확장성 | TC 없음 | 시나리오 자동 전환 구조 + 아키텍처 문서로 검증 |
| REQ-NF-007 안전성 | TC 없음 | UI 교육목적 문구 + README 명시 + 인터락 로직으로 보강 |
