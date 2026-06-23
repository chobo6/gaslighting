# G-SAFE Infrastructure Guide

## 프로젝트 개요

G-SAFE는 가스 누출을 감지하고 위험도를 예측하여 자동으로 차단 및 복구를 수행하는 안전 관리 시스템이다.

본 문서는 프로젝트의 인프라 환경과 운영 방향을 정의하기 위해 작성되었다.

---

## 개발 환경

| 항목     | 버전                         |
| ------ | -------------------------- |
| Python | 3.11.x                     |
| Unity  | 2022.3 LTS                 |
| Git    | 최신 버전                      |
| OS     | Windows 10 / 11            |
| Cloud  | AWS EC2 (Rocky Linux 9 예정) |

---

## 브랜치 전략

* main : 최종 배포 및 발표용
* develop : 통합 개발 브랜치
* feature/* : 기능 개발 브랜치

예시

* feature/infra
* feature/backend
* feature/frontend
* feature/ml

---

## 배포 환경

### 개발 환경

* SQLite
* FastAPI
* Unity

### 운영 환경(예정)

* AWS EC2
* Rocky Linux 9
* FastAPI
* PostgreSQL

---

## 인프라 담당 업무

* 개발 환경 표준화
* Git 저장소 관리
* 브랜치 전략 관리
* AWS 서버 환경 구성
* 배포 환경 문서화
* 운영 환경 관리

---

## 향후 계획

1. AWS EC2 서버 구성
2. FastAPI 배포 환경 구축
3. PostgreSQL 적용 검토
4. Docker 적용 검토
5. 모니터링 환경 구축 검토
