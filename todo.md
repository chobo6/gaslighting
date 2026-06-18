# G-SAFE 프로젝트 TODO

> 기준일: 2026-06-18 | 전체 일정: 2026.06.16 – 07.13 (4주)
> 목표 지표: R² > 0.9 · 오탐률 < 10% · Recall > 0.9 · API 응답 < 1s · 30fps+

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

### 기획 잔여 (선택사항)

- [ ] **와이어프레임 화면 보완** (`presentation_v2/wireframe.html` 탭 추가)
  - [ ] 대시보드 화면 — 시스템 상태 요약 / 최근 이력 / 모델 성능 카드
  - [ ] 시나리오 선택 화면 — 연구소 / 개발실 / 배관실 카드 + 설정값

---

## 개발 단계 — WEEK 02 (06.23 – 06.30)

### 데이터 / ML — 담당: 김응석

- [ ] **UCI 데이터셋 다운로드 및 검증**
  - [ ] [UCI Gas Sensor Array Drift Dataset](https://archive.ics.uci.edu/ml/datasets/Gas+Sensor+Array+Drift+Dataset) 다운로드
  - [ ] 컬럼 확인: `time`, `sensor_1~16`, `gas_concentration` 누락 없음 → TC-DATA-001
  - [ ] 전체 행 수 · 결측값 · 이상치 분포 확인 후 `data/eda_report.txt` 저장

- [ ] **전처리 스크립트** (`data/preprocess.py`)
  - [ ] 결측값 선형 보간 (또는 행 제거 기준 명시)
  - [ ] `MinMaxScaler`로 `sensor_1~16` 정규화, `scaler.pkl` 저장
  - [ ] 슬라이딩 윈도우 (window=10, step=1) 적용해 시계열 피처 생성
  - [ ] 전처리 결과 `data/processed.csv` 저장

- [ ] **라벨 생성 함수** (`data/label_generator.py`)
  - [ ] 1단계: `gas_concentration` 임계값(예: LEL 25%) 기준 기본 점수 산출
  - [ ] 2단계: 1초 slope(`np.gradient`) 가중치 부여 (급등 시 +보정)
  - [ ] 3단계: 위험 지속 시간(연속 초과 카운트) 패널티 적용
  - [ ] 최종 `risk_score` 0~100 클리핑 → TC-DATA-002
  - [ ] 라벨 분포 히스토그램 출력해 치우침 확인

- [ ] **데이터 분리**
  - [ ] Train 70% / Validation 15% / Test 15% 분리 (`random_state=42`)
  - [ ] 분리 결과 행 수 출력 후 `data/splits/` 저장

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

- [ ] **위험률 시계열 그래프** (`Assets/Scripts/RiskGraph.cs`)
  - [ ] 최근 60초 데이터 `Queue<float>` 유지
  - [ ] `LineRenderer` 또는 UI `RawImage` + `Texture2D` 방식 구현
  - [ ] threshold 점선: 별도 `LineRenderer` 고정 y값
  - [ ] 차단 이벤트 발생 시 마커 `◆` Sprite spawn

- [ ] **CSV 센서 재생 스크립트** (`Assets/Scripts/SensorPlayer.cs`)
  - [ ] `Resources.Load<TextAsset>` 또는 `StreamingAssets` 경로로 CSV 로드
  - [ ] `InvokeRepeating(1.0f)` 1초 주기 타임스텝 진행
  - [ ] 각 행의 `sensor_1~16` 파싱 → `float[]` 로 `ApiClient.cs`에 전달

---

## 개발 단계 — WEEK 03 (06.30 – 07.06)

### ML 모델 학습 — 담당: 김응석

- [ ] **RandomForest 학습** (`model/train_rf.py`)
  - [ ] `n_estimators=200, max_depth=10, random_state=42` 기본값으로 시작
  - [ ] `GridSearchCV` 또는 수동 튜닝으로 최적값 탐색
  - [ ] MAE / RMSE / R² 출력 → R² > 0.9 목표 미달 시 파라미터 재조정

- [ ] **GBR 학습** (`model/train_gbr.py`)
  - [ ] `learning_rate=0.05, n_estimators=300, subsample=0.8`
  - [ ] RF vs GBR 성능 비교 표 콘솔 출력
  - [ ] 더 좋은 모델 `model/best_model.pkl` 로 저장 (joblib)

- [ ] **Threshold 탐색** (`model/threshold_search.py`)
  - [ ] 검증 데이터로 후보 범위 60~95 (step=1) 전체 스캔
  - [ ] 각 후보별 `Recall`, `F2-score`, 오탐률(`FPR`), 차단 시점 오차(초) 산출
  - [ ] 필터: 오탐률 < 0.10 AND Recall > 0.90 조건 만족 구간 추출
  - [ ] 조건 만족 구간 중 차단 시점 오차 최소값 선택 → TC-ML-002
  - [ ] 결과를 `model/threshold_report.csv` 저장

### FastAPI 서버 — 담당: 홍제규

- [ ] **프로젝트 구조 생성** (`backend/`)
  ```
  backend/
  ├── main.py              # 앱 초기화, CORS (origins=["*"])
  ├── routers/
  │   ├── predict.py       # POST /api/predict-risk
  │   ├── logs.py          # POST /api/logs/cutoff
  │   └── meta.py          # GET /api/health, /scenarios, /model-performance
  ├── models/
  │   └── schemas.py       # Pydantic 요청/응답 스키마
  ├── db/
  │   ├── database.py      # SQLite 연결 (SQLAlchemy)
  │   └── init_db.py       # 테이블 생성 스크립트
  └── ml/
      └── predictor.py     # 모델 로드 & predict() 래퍼
  ```
- [ ] **`POST /api/predict-risk`** (TC-API-001)
  - [ ] `schemas.py`에 `SensorInput` (sensor_1~16 float, scenario_id int) Pydantic 모델 정의
  - [ ] `predictor.py`에서 `scaler.transform()` → `model.predict()` 순서 적용
  - [ ] `risk_level` 변환: < 40 → 정상, 40~69 → 주의, 70~threshold → 위험, ≥ threshold → 차단
  - [ ] 응답에 `response_ms` 포함 (시작~종료 `time.time()` 차이)
- [ ] **입력 검증** (TC-API-002)
  - [ ] Pydantic `Field(ge=0.0, le=1.0)` 범위 검증 (정규화 후 값 기준)
  - [ ] 필수 필드 누락 시 FastAPI 자동 HTTP 422 반환 확인
- [ ] **`GET /api/health`**
  - [ ] 앱 시작 시 `predictor.py` 모델 로드 → `model_loaded` 플래그 설정
  - [ ] `db_connected` SQLAlchemy `engine.connect()` 예외 여부로 판단
- [ ] **`GET /api/scenarios`** — `scenarios` 테이블 전체 조회 반환
- [ ] **`GET /api/model-performance`** — `model_results` 테이블에서 RF/GBR 지표 반환
- [ ] **`POST /api/logs/cutoff`** (TC-DB-002) — `cutoff_events` INSERT 후 `event_id` 반환
- [ ] **DB 초기화** (`db/init_db.py`)
  - [ ] `gas_readings`, `cutoff_events`, `scenarios`, `sensor_configs`, `model_results` 5개 테이블 CREATE
  - [ ] `scenarios` 초기 데이터 INSERT (연구소/개발실/배관실 3건)
- [ ] **실행 테스트**: `uvicorn main:app --reload` 후 Swagger UI(`/docs`) 전 엔드포인트 수동 확인

### Unity-API 연동 — 담당: 홍제규 + 안세웅

- [ ] **`ApiClient.cs`** (`Assets/Scripts/ApiClient.cs`)
  - [ ] `UnityWebRequest.Post(url, jsonBody)` 비동기 코루틴 구현
  - [ ] `SensorPlayer.cs`에서 1초마다 `StartCoroutine(RequestRisk(sensorValues))` 호출
  - [ ] JSON 응답 파싱: `JsonUtility.FromJson<RiskResponse>()` → `RiskGraphManager` 전달
  - [ ] 타임아웃 2초 설정, 실패 시 이전 `risk_score` 유지 (fallback · REQ-NF-006)
  - [ ] 3회 연속 실패 시 로컬 데모 모드 자동 전환 (API 의존성 제거)
- [ ] **연동 통합 테스트** (TC-INT-001)
  - [ ] FastAPI 서버 로컬 실행 확인 (`GET /api/health` → 200)
  - [ ] Unity Play Mode에서 API 호출 → risk_score HUD 반영 확인
  - [ ] Network 탭(또는 Unity Console)에서 응답 시간 < 1s 확인

---

## 테스트 단계 — WEEK 04 (07.06 – 07.13)

### 단위 테스트 — 담당: 이성룡

| TC ID | 항목 | 수행일 | 담당 |
|-------|------|--------|------|
| TC-DATA-001 | UCI 데이터 필수 컬럼 확인 | 07-06 | 김응석 |
| TC-DATA-002 | risk_score 0~100 범위 검증 | 07-06 | 김응석 |
| TC-ML-001 | RF/GBR 학습 성공 + 지표 출력 | 07-07 | 김응석 |
| TC-ML-002 | Threshold 탐색 최적값 선정 | 07-07 | 김응석 |
| TC-API-001 | POST /predict-risk 정상 응답 | 07-08 | 홍제규 |
| TC-API-002 | 잘못된 입력 → HTTP 422 반환 | 07-08 | 홍제규 |
| TC-DB-001 | 시뮬레이션 후 gas_readings 저장 | 07-08 | 장영재 |
| TC-DB-002 | threshold 초과 → cutoff_events 저장 | 07-08 | 장영재 |
| TC-UNITY-001 | Particle System 누출 지점 활성화 | 07-09 | 안세웅 |
| TC-UNITY-002 | risk_score ≥ threshold → 밸브/경보/환기팬 동작 | 07-09 | 안세웅 |
| TC-UNITY-003 | HUD 위험률 1초 이하 주기 갱신 | 07-09 | 장영재 |

### 통합 테스트 — 담당: 이성룡

| TC ID | 항목 | 수행일 |
|-------|------|--------|
| TC-INT-001 | Unity → API → HUD 실시간 반영 | 07-10 |
| TC-INT-002 | 정상 시나리오 (risk < threshold) — 차단 미발생 | 07-10 |
| TC-INT-003 | 위험 시나리오 (risk ≥ threshold) — 자동차단 + DB 저장 | 07-10 |
| TC-PRES-001 | 발표용 Demo 자동 흐름 중단 없이 완료 | 07-13 |

### 발표 준비 — 담당: 전체

- [ ] 트러블슈팅 문서 실제 이슈로 업데이트 (`presentation_v2/troubleshooting.html`)
- [ ] 팀원 회고 최종 작성 — 기획 발표 후 각자 작성
- [ ] 발표 슬라이드 최종 확인 (내용 변경 사항 반영)
- [ ] 발표 환경 사전 테스트 — 30fps+ 유지, 해상도 1920×1080 확인
- [ ] 최종 GitHub push + 버전 태그 (`git tag v1.0.0`)

---

## RTM 미커버 요구사항 처리 계획

| 요구사항 | 이유 | 처리 방법 |
|----------|------|-----------|
| REQ-F-011 위험 원인 설명 | TC 없음 | WEEK 04 결과 화면 구현 후 수동 확인 |
| REQ-F-013 수동 파라미터 조정 | TC 없음 | WEEK 04 슬라이더 UI 구현 후 수동 확인 |
| REQ-NF-005 확장성 | TC 없음 | 아키텍처 문서(시스템 아키텍처 슬라이드)로 검증 |
| REQ-NF-007 안전성 | TC 없음 | UI 교육목적 문구 + README 명시로 대체 |
