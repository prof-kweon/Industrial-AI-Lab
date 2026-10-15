# Industrial AI Lab

> **From-Scratch AI Research → Student Applications → Industrial AI Platform**

**기간:** 2026년 가을학기 ~ 2027년 여름방학 (1차 계획)  
**방식:** 1인 핵심 원리 습득·직접 구현 → 이해도 검증 → 학생들과 산업 응용  
**구성:** Track A · Track B · Common Foundation

## 아키텍처 한눈에 보기

```text
Track A: Cognitive AI          Track B: Perception & Physical AI
Agent | RAG | Small LLM         Vision | Voice | Time-Series
Document AI                    Reinforcement Learning
              \              /
           Common APIs & Event Contracts
                       |
        Multi-Modal Data & MLOps Platform
                       |
        Industrial AI Integrated Prototype
```
## 프로젝트 개요

1인 연구개발로 8개 AI 기술의 핵심 원리를 직접 구현하고 검증한 뒤, 충분한 이해를 확보한 모듈을 학생들과 산업 응용 및 플랫폼화로 발전시킨다. 2026년 가을학기부터 2027년 여름방학까지를 1차 계획 기간으로 설정한다.

## 핵심 원칙

Understand → Implement from Scratch → Experiment & Validate → Mastery Gate → Apply with Students. NumPy/PyTorch로 교육용 최소 구현을 먼저 작성하고, 오픈소스 구현과 비교한 뒤 산업용 모듈로 확장한다. 이해도 100%는 무제한 지식이 아니라 원리 설명, 재구현, 변경, 디버깅, 실험 설계 능력으로 운영상 정의한다.

## Track A — Cognitive AI Platform

1. AI Agent: ReAct, planning, tool calling, memory, 실행 상태 및 실패 복구.
2. RAG: chunking, embedding, BM25, dense/hybrid retrieval, reranking, 인용·평가.
3. Small LLM: tokenizer, attention, Transformer, 미니 GPT 학습, LoRA/양자화 비교.
5. Document AI: OCR, layout, 표 구조 추출, 도면·이미지 이해, 문서 기반 질의응답.

## Track B — Perception & Physical AI Platform

4. Vision AI: CNN/ViT, 객체 검출, 불량·이상 탐지.
6. Voice AI: STFT, mel spectrogram, CTC, ASR, 음성 명령 인터페이스.
7. Time-Series AI: 예측, 이상 탐지, 센서 융합, drift 감지.
8. Reinforcement Learning: MDP, Bellman, DQN/PPO, 시뮬레이터 기반 정책 평가. 실제 설비 제어는 안전 검증과 승인 절차 이후 별도 단계로 진행한다.

## Common Foundation — Multi-Modal Data & MLOps Platform

Track A/B가 공유하는 독립적인 공통 플랫폼으로 설계한다. 수집·검증·저장·정제·동기화·라벨링·데이터셋 생성·버전 관리·모델 등록·서빙·모니터링을 지원한다. 겨울 Prototype 단계부터 원본 데이터, 추론 결과, 실행 로그와 피드백을 수집한다.

## 지원 데이터 타입

Text/Logs, PDF/DOCX/PPTX/표, Image/도면, Video, Audio/설비 소음, Time-Series/센서, Structured(CSV/JSON/SQL), Event/Stream(MQTT 등), Spatial/3D/Robot Pose, Simulation/RL Trajectory. 우선 구현: 텍스트·문서·이미지·음성·시계열·구조화 데이터. 이후 비디오·스트림·3D·시뮬레이션을 단계적으로 확장한다.

## 데이터 파이프라인 설계

Sources → Batch/Streaming Ingestion → Schema Validation & Quality → Raw Zone → Processed Zone → Curated Zone → Dataset/Feature/Vector Interfaces → Training/Evaluation/Inference → Feedback. 공통 메타데이터: data_id, source_id, modality, asset_id, event_time, ingest_time, schema_version, dataset_version, provenance, license, access_policy. 멀티모달 데이터는 설비 ID 및 시간 기준으로 연결하며 시계 오차·결측·샘플링 주기 차이를 관리한다. 원본 불변 보관, lineage, 재처리, 접근 통제, 개인정보 보호 및 보존 정책을 포함한다.

## 개발 일정

2026.10–12 가을학기: 핵심 논문/수식 학습, 환경 구축, SRS, 아키텍처, 데이터 계약, 평가 설계.
2026.12–2027.02 겨울방학: 8개 핵심 기술의 교육용 최소 구현과 Track A/B Prototype v0.1, 초기 데이터·로그 수집.
2027.03–06 봄학기: 미통과 기술 보완, 멀티모달 batch/stream 파이프라인, 데이터셋·실험 버전 관리, 모델 개선.
2027.07–08 여름방학: 학생 참여 산업 응용, API/배포/관측성/보안/성능 검증, Platform v1.0 후보. 실제 상용 배포는 별도 현장 검증 후 결정한다.

## Mastery Gate — 기술별 완료 기준

① 수학적·알고리즘적 원리를 독립적으로 설명할 수 있다. ② 주요 핵심 루프를 참고 코드 없이 재구현할 수 있다. ③ 단위 테스트와 작은 데이터 실험을 통과한다. ④ 베이스라인 대비 정확도·지연시간·메모리 등을 정량 비교한다. ⑤ 실패 사례를 설명하고 최소 1개 ablation/변형 실험을 수행한다. ⑥ 다른 사람에게 설명 가능한 문서와 데모를 남긴다. 통과 전에는 해당 모듈을 학생 응용의 안정된 기반으로 간주하지 않는다.

## 공통 산업 시나리오

스마트 제조 설비의 이상 탐지·원인 분석·의사결정 지원을 공통 데모로 채택한다. Track B가 영상/센서로 이상을 감지하고 Track A가 매뉴얼과 로그를 검색해 원인과 대응 방안을 제시한다. 강화학습은 시뮬레이션에서 정책을 비교하며, 현장 실행은 사람의 승인과 별도 안전 절차를 전제로 한다.

## 평가 체계

RAG: Recall@k, MRR, groundedness. Agent: task success, tool error, recovery rate. LLM: loss/perplexity, throughput, VRAM. Document: field/table extraction F1. Vision: AUROC/mAP. Voice: WER, latency. Time-Series: MAE/F1/false alarm rate. RL: episode return, success rate, safety violations. Data: freshness, completeness, schema failure, pipeline latency, lineage coverage. 목표 수치는 초기 baseline 측정 후 확정한다.

## 저장소 구조

track-a/{agent,rag,small-llm,document-ai}/; track-b/{vision,voice,time-series,reinforcement-learning}/; foundation/{ingestion,schemas,processing,storage,datasets,mlops}/; apps/demo/; docs/{architecture,requirements,design,evaluation}/; experiments/; tests/; infra/. 각 기술 디렉터리에 Theory.md, FromScratch.py, Experiments.ipynb, Benchmark.md, Mastery_Checklist.md를 둔다.

## 운영 및 위험 관리

1인 개발 범위를 고려하여 작은 모델·작은 데이터·명확한 테스트로 시작한다. 겨울에는 8개 기술의 최소 구현과 제한된 통합 데모를 목표로 하며, 상용 품질을 약속하지 않는다. 데이터 사용 권한, 보안, 비용, GPU 자원, 데이터 품질, 모델 환각, 로봇 제어 안전성, 일정 초과를 위험 항목으로 추적한다.

## 주요 산출물

가을: Project Charter, SRS, Architecture, Module Design, Data Architecture, Evaluation Plan. 겨울: 8개 핵심 구현, 테스트, 실험 노트, Prototype v0.1. 봄: Multi-Modal Data Pipeline, 데이터 카탈로그·버전 관리, 개선 실험. 여름: 학생 응용 데모, 운영·배포 체계, Platform v1.0 후보, 기술 보고서 및 GitHub 공개.

## 상세 설계 문서

- [System Architecture Design](docs/architecture/System_Architecture_Design.md)
- [Word 버전](docs/architecture/System_Architecture_Design.docx)
