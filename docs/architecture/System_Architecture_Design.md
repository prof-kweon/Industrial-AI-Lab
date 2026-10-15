# System Architecture Design

Industrial AI Lab | Draft v0.1 | 2026-10-15

```text
Industrial Data Sources (Documents, Images, Audio, Sensors, Events, Simulation)
                           |
                 Batch / Stream Ingestion
                           |
                Validation & Data Quality
                           |
                 Raw -> Processed -> Curated
                           |
                  Datasets / Features
                  /                  \
       Track A: Cognitive       Track B: Physical
       Agent/RAG/LLM/Docs       Vision/Voice/TS/RL
                  \                  /
                  Shared APIs & Events
                           |
                 Industrial AI Demo
                           |
                 Monitoring & Feedback
```

## 1. 목표와 설계 원칙

1인 개발자가 8개 기술의 핵심 원리를 From Scratch로 구현·검증하고, Mastery Gate 통과 후 학생들과 산업 응용을 진행한다. 모듈형 모놀리스로 시작하고 공통 API/이벤트 계약을 정의한다. 원본 불변 보존, 실험 재현성, 인간 승인 기반 제어를 기본 원칙으로 한다.

## 2. Track A — Cognitive AI Platform

Agent: 계획·도구 호출·실행 상태·재시도. RAG: 청킹·BM25·벡터 검색·재정렬·근거 추적. Small LLM: 토크나이저·Transformer·학습·경량화. Document AI: OCR·레이아웃·표·도면 분석. 교육용 핵심 구현과 산업용 통합 코드를 분리한다.

## 3. Track B — Perception & Physical AI Platform

Vision: 영상·이미지 불량/이상 탐지. Voice: STFT·Mel·CTC·ASR 및 음성 명령. Time-Series: 센서 예측·이상 탐지·drift. RL: MDP·DQN/PPO·시뮬레이터 정책 평가. 실제 제어는 별도의 안전 검증과 인간 승인 후 추진한다.

## 4. Common Foundation — Multi-Modal Data & MLOps

Batch/Streaming 수집 → 스키마 검증·품질 검사 → Raw → Processed → Curated → 데이터셋/피처/벡터 인덱스 → 학습·평가·서빙 → 피드백. 데이터 카탈로그·계보·버전·모델 레지스트리·실험 추적·모니터링을 공통 서비스로 제공한다.

## 5. 지원 데이터 타입

Text/Log, PDF/DOCX/PPTX, Image/도면, Video, Audio/소음, Time-Series/센서, CSV/JSON/SQL, Event/Stream, Spatial/3D/Robot Pose, Simulation/RL Trajectory. 1차 범위는 텍스트·문서·이미지·음성·시계열·구조화 데이터, 이후 영상·스트림·3D·시뮬레이션으로 확장한다.

## 6. 공통 데이터 계약

필수 필드: data_id, source_id, modality, schema_version, event_time(UTC), ingest_time(UTC), uri, checksum. 선택 필드: asset_id, correlation_id, dataset_version, provenance, license, access_policy, retention_policy, quality_status. 설비 ID와 시간창으로 모달리티를 연결하고 시계 오차·결측·샘플링 차이를 기록한다.

## 7. 서비스와 이벤트 인터페이스

API 초안: /ingest, /datasets, /retrieve, /infer, /agents/runs, /models, /health. 이벤트 초안: data.ingested.v1, data.validated.v1, inference.completed.v1, anomaly.detected.v1. request_id, trace_id, schema_version, 멱등성 키, 재시도·실패 격리 정책을 명시한다.

## 8. 저장소 및 기술 선택

초기 개발은 파일/객체 저장소, SQLite/PostgreSQL 메타데이터, Parquet 데이터셋, FastAPI, Docker Compose를 기본 후보로 한다. 검색·스트림·시계열 특화 저장소는 실험 요구량을 측정한 후 선택한다. 성능 근거가 없는 초기 마이크로서비스 분리는 지양한다.

## 9. 데이터 품질·보안·운영

완전성·신선도·중복·스키마 오류·시간 정렬·라인리지 커버리지를 측정한다. 데이터 사용권, 개인정보 비식별화, 최소 권한, 비밀정보 관리, 보존/삭제, 감사 로그, 백업·복구를 설계한다. 로그·메트릭·트레이스와 모델 성능/drift를 추적한다.

## 10. 평가 및 Mastery Gate

기술별 Theory.md, FromScratch.py, Experiments.ipynb, Benchmark.md, Mastery_Checklist.md를 남긴다. 원리 설명·독립 재구현·단위 테스트·베이스라인 비교·실패 분석·변형 실험을 완료해야 학생 응용용 안정 모듈로 승격한다.

## 11. 학사 일정별 개발 계획

2026 가을: 학습·환경·SRS·아키텍처·데이터 계약·평가 설계. 2026~27 겨울: 8개 최소 구현, Track A/B Prototype v0.1, 원본/추론/오류 로그 수집. 2027 봄: 데이터 파이프라인·버전 관리·재학습 실험. 2027 여름: 학생 응용·API·배포·관측성·보안·성능·장애 복구 검증. 상용 현장 배포는 별도 검증 이후 결정한다.

## 12. 아키텍처 결정 및 다음 단계

확정: Two-Track+Common Foundation, 공통 제조 설비 이상 진단 시나리오, 모듈형 모놀리스, 승인 기반 제어. 미정: GPU/예산, 첫 데이터셋, 처리량/지연 SLO, 스트리밍 브로커, 저장소 규모. 다음 산출물: SRS, JSON Schema, OpenAPI, Prototype 수직 슬라이스, ADR.
