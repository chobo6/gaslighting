# G-SAFE 프로젝트 TODO

> 기준일: 2026-06-18 | 전체 일정: 2026.06.16 – 07.13 (4주)

---

## 기획 단계 (WEEK 01 | 06.16 – 06.23)

### ✅ 완료

- [x] 요구사항 명세서 작성 (`presentation_v2/requirements.html`) — 기능 17 + 비기능 8 = 25개
- [x] WBS 작성 (`presentation_v2/wbs.html`) — 36 tasks, 4주 Gantt
- [x] 메뉴 트리 & 사용자 시나리오 (`presentation_v2/menu_scenario.html`) — 4단계 메뉴 + 4 시나리오
- [x] 와이어프레임 작성 (`presentation_v2/wireframe.html`) — 시뮬레이션 화면 + 결과 상세 2화면
- [x] 스토리보드 작성 (`presentation_v2/storyboard.html`) — 5 Scene (정상→누출→주의→차단→결과)
- [x] 테이블 정의서 / ERD (`presentation_v2/table_definition.html`) — 5개 테이블 정의
- [x] 테스트 케이스 작성 (`presentation_v2/test_cases.html`) — 15개 TC (Data/ML/Backend/Unity/Integration)
- [x] 트러블슈팅 & 회고 초안 (`presentation_v2/troubleshooting.html`) — 예상 이슈 4건, 팀원 회고 초안
- [x] 발표 슬라이드 제작 (`presentation_v2/index.html`) — 9슬라이드 (표지 포함)
- [x] GitHub 연결 & 초기 push (`https://github.com/chobo6/gaslighting.git`)
- [x] README.md 작성 (`README.md`) — 프로젝트 개요 / 아키텍처 / API / 팀 / 일정

### ⬜ 미완료 (기획 단계 잔여)

- [x] **API 명세서 작성** (`presentation_v2/api_spec.html`) — 담당: 홍제규
  - [x] `GET /api/health` — 서버 상태 확인 (request/response/error 포함)
  - [x] `POST /api/predict-risk` — sensor_1~16 입력 → risk_score + risk_level 반환
  - [x] `GET /api/model-performance` — RF/GBR 모델별 MAE, RMSE, R², threshold 조회 (REQ-F-017)
  - [x] `GET /api/scenarios` — 시나리오 목록 (연구소/개발실/배관실) 반환
  - [x] `POST /api/logs/cutoff` — 자동차단 이벤트 cutoff_events 테이블에 저장 (REQ-F-015)
  - [x] 공통 에러 응답 형식 정의 (HTTP 422, 500, 503)
  - [x] 응답 시간 목표 명시 (< 1s)

- [x] **요구사항 추적 매트릭스(RTM) 추가** (`presentation_v2/requirements.html` 탭 추가)
  - [x] 요구사항 ID (REQ-F-001~017, REQ-NF-001~008) ↔ TC ID 매핑 표 작성
  - [x] 커버리지 집계: 25개 중 TC 커버 21개 (84%), 미커버 4개, BY DOCS 1개
  - [x] 미커버 요구사항 명시 — REQ-F-011(위험원인설명), REQ-F-013(수동파라미터), REQ-NF-005(확장성), REQ-NF-007(안전성)

- [ ] **와이어프레임 화면 보완** (선택사항) (`presentation_v2/wireframe.html` 탭 추가)
  - [ ] 대시보드 화면 (시스템 상태 요약 / 최근 이력 / 모델 성능)
  - [ ] 시나리오 선택 화면 (연구소 / 개발실 / 배관실 카드)

---

## 개발 단계 (WEEK 02 | 06.23 – 06.30)

### 데이터 / ML — 담당: 김응석

- [ ] UCI Gas Sensor Array 데이터셋 다운로드 및 확인
  - [ ] 컬럼 확인: time / sensor_1~16 / gas_concentration 누락 여부 (TC-DATA-001)
  - [ ] 전체 행 수, 결측값, 이상치 분포 파악
- [ ] 데이터 전처리 스크립트 작성 (`data/preprocess.py`)
  - [ ] 결측값 처리 (보간 또는 제거)
  - [ ] sensor_1~16 정규화 (MinMaxScaler 또는 StandardScaler)
  - [ ] 시계열 슬라이딩 윈도우 적용
- [ ] risk_score 라벨 생성 함수 작성 (`data/label_generator.py`)
  - [ ] 1단계: 농도(ppm) 임계값 기반 기본 점수 산출
  - [ ] 2단계: 증가 속도(slope) 가중치 부여
  - [ ] 3단계: 지속 시간 패널티 적용
  - [ ] 출력 범위 0~100 검증 (TC-DATA-002)
- [ ] 학습/검증/테스트 데이터 분리 (8:1:1 또는 7:2:1)

### Unity / 시뮬레이션 — 담당: 안세웅

- [ ] Unity 프로젝트 생성 및 GitHub 연동 (`.gitignore` Unity 설정)
- [ ] 3D 연구소 씬 구성
  - [ ] 배관, 밸브 오브젝트 배치
  - [ ] 누출 지점 빈 오브젝트(Empty) 설정
  - [ ] 카메라 배치 및 시점 조작 설정
- [ ] Particle System 설정 (가스 확산 효과)
  - [ ] 누출 지점에서 Emission 활성화/비활성화 제어
  - [ ] 색상·속도·수명 파라미터 조정
- [ ] 밸브/경보/환기팬 애니메이션
  - [ ] 밸브 OPEN ↔ CLOSED 상태 전환 애니메이션
  - [ ] 경보등 ON/OFF 깜빡임 이펙트
  - [ ] 환기팬 회전 애니메이션

### HUD / UI — 담당: 장영재

- [ ] Unity HUD 구성 (Canvas - Overlay)
  - [ ] 위험률 게이지 바 (0~100%)
  - [ ] threshold 기준선 표시
  - [ ] 위험 등급 배지 (정상🟢 / 주의🟡 / 위험🔴 / 차단완료⚫)
  - [ ] 밸브/경보/환기 상태 아이콘
- [ ] 위험률 시계열 그래프 (최근 60초)
  - [ ] LineRenderer 또는 UI Shader 기반 구현
  - [ ] threshold 점선 표시
  - [ ] 차단 이벤트 마커(◆) 표시
- [ ] CSV 센서 데이터 재생 스크립트 (`SensorPlayer.cs`)
  - [ ] 1초 단위 타임스텝 재생
  - [ ] sensor_1~16 값 HUD에 갱신

---

## 개발 단계 (WEEK 03 | 06.30 – 07.06)

### ML 모델 학습 — 담당: 김응석

- [ ] RandomForest Regressor 학습 (`model/train_rf.py`)
  - [ ] n_estimators, max_depth 하이퍼파라미터 설정
  - [ ] MAE / RMSE / R² 평가 출력
  - [ ] R² > 0.9 달성 여부 확인 (목표 지표)
- [ ] Gradient Boosting Regressor 학습 (`model/train_gbr.py`)
  - [ ] learning_rate, n_estimators 튜닝
  - [ ] RF vs GBR 성능 비교 표 출력
- [ ] Threshold 탐색 스크립트 (`model/threshold_search.py`)
  - [ ] 후보 범위: 60~95 스캔
  - [ ] 각 후보별 Recall, F2-score, 오탐률, 차단 시점 오차 산출
  - [ ] 조건: 오탐률 < 10%, Recall > 0.9 만족 구간 필터링
  - [ ] 최적 threshold 선정 및 결과 저장 (TC-ML-002)
- [ ] 모델 저장 (`model/model.pkl`, `model/scaler.pkl`) — joblib

### FastAPI 서버 — 담당: 홍제규

- [ ] FastAPI 프로젝트 구조 생성 (`backend/`)
  - [ ] `main.py` — 앱 초기화, CORS 설정
  - [ ] `routers/predict.py` — 예측 라우터
  - [ ] `routers/logs.py` — 로그 저장 라우터
  - [ ] `models/schemas.py` — Pydantic 스키마 정의
  - [ ] `db/database.py` — SQLite 연결 설정
- [ ] `POST /api/predict-risk` 구현 (TC-API-001)
  - [ ] sensor_1~16 수신 → scaler 적용 → model.predict() 호출
  - [ ] risk_score 0~100 반환, risk_level 문자열 변환
  - [ ] 응답 시간 < 1s 확인
- [ ] `GET /api/model-performance` 구현
  - [ ] model_results 테이블에서 RF/GBR 성능 지표 반환
- [ ] `GET /api/health` 구현 — 서버 상태 200 OK 반환
- [ ] 입력 검증 (TC-API-002)
  - [ ] 필수 필드 누락 시 HTTP 422 반환
  - [ ] 범위 이탈 값 처리
- [ ] DB 스키마 생성 스크립트 (`db/init_db.py`)
  - [ ] gas_readings 테이블 생성
  - [ ] cutoff_events 테이블 생성
  - [ ] model_results 테이블 생성

### Unity-API 연동 — 담당: 홍제규 + 안세웅

- [ ] `ApiClient.cs` 스크립트 작성
  - [ ] `UnityWebRequest` 비동기 코루틴 구현
  - [ ] POST JSON 전송 (sensor_1~16 payload)
  - [ ] 응답 파싱 → risk_score HUD 갱신
  - [ ] API 실패 시 이전 risk_score 유지 (fallback, REQ-NF-006)
- [ ] 연동 테스트 (TC-INT-001)
  - [ ] FastAPI 로컬 서버 실행 → Unity에서 API 호출
  - [ ] risk_score HUD 즉시 반영 확인

---

## 테스트 단계 (WEEK 04 | 07.06 – 07.13)

### 단위 테스트 — 담당: 이성룡

- [ ] TC-DATA-001: UCI 데이터 필수 컬럼 확인 (07-06)
- [ ] TC-DATA-002: risk_score 범위 0~100 검증 (07-06)
- [ ] TC-ML-001: RF/GBR 회귀 학습 성공 + 지표 출력 확인 (07-07)
- [ ] TC-ML-002: Threshold 탐색 — F2 기준 최적값 선정 확인 (07-07)
- [ ] TC-API-001: `POST /api/predict-risk` 정상 응답 확인 (07-08)
- [ ] TC-API-002: 잘못된 입력 시 HTTP 422 반환 확인 (07-08)
- [ ] TC-DB-001: 시뮬레이션 후 gas_readings 저장 확인 (07-08)
- [ ] TC-DB-002: threshold 초과 시 cutoff_events 저장 확인 (07-08)
- [ ] TC-UNITY-001: Particle System 누출 지점 활성화 확인 (07-09)
- [ ] TC-UNITY-002: risk_score ≥ threshold 시 밸브/경보/환기팬 동작 확인 (07-09)
- [ ] TC-UNITY-003: HUD 위험률 1초 이하 주기 갱신 확인 (07-09)

### 통합 테스트 — 담당: 이성룡

- [ ] TC-INT-001: Unity → API → HUD 실시간 반영 확인 (07-10)
- [ ] TC-INT-002: 정상 시나리오 (risk < threshold) — 차단 미발생 확인 (07-10)
- [ ] TC-INT-003: 위험 시나리오 (risk ≥ threshold) — 자동차단 + DB 저장 확인 (07-10)
- [ ] TC-PRES-001: 발표용 Demo 자동 흐름 (누출→차단) 중단 없이 완료 (07-13)

### 발표 준비 — 담당: 전체

- [ ] 트러블슈팅 문서 실제 이슈로 업데이트 (`presentation_v2/troubleshooting.html`)
- [ ] 팀원 회고 최종 작성 (기획 발표 후)
- [ ] 발표 슬라이드 내용 최종 확인 및 수정
- [ ] 발표 환경 사전 테스트 (30fps 이상 유지 확인)
- [ ] 최종 GitHub push 및 버전 태그 설정

---

## 비고

- **목표 지표**: R² > 0.9 · 오탐률 < 10% · Recall > 0.9 · 응답 < 1s · 30fps+
- **발표일**: 2026.07.13
- **미완료 기획 항목**: API 명세서, RTM — 개발 착수 전 완료 권장
