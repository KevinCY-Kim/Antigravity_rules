# KevinCY-Kodex: AI Engineering Standard & Playbook

**Version:** 2025.11.27 (Universal Hybrid Edition)
**Author:** KevinCY-Kim (System Architect)
**Compatible With:** GPT KODEX, Google Antigravity

---

## 1. 소개 (Introduction)

`KevinCY-Kodex`는 확장 가능한 AI 애플리케이션(RAG, Voice Bot)을 구축하기 위해 설계된 **Unified Engineering Standard(통합 엔지니어링 표준)**입니다.
**Clean Architecture**, **Hybrid AI Strategy**, **Agent Safety**를 기반으로, 기획부터 배포까지의 개발 프로세스를 체계화한 플레이북을 제공합니다.

이 문서는 **개발팀**뿐만 아니라, **AI 에이전트(Antigravity)**가 프로젝트 컨텍스트를 이해하고 일관성 있는 코드를 작성하기 위한 **핵심 지침서(Root Instruction)** 역할을 합니다.

---

## 2. 핵심 철학 (Core Philosophy)

우리는 다음 3가지 원칙을 기반으로 개발합니다.

1.  **Clean Architecture (Universal):**
    -   도메인(예: 공공, 금융)이 바뀌어도 변하지 않는 견고한 구조.
    -   `Router` -> `Service` -> `Repository/AI` 계층의 엄격한 분리.
2.  **Hybrid AI Strategy (Cost & Privacy):**
    -   **API (SKT A.X)**와 **Local (Ollama)** 모델을 상황에 맞춰 유동적으로 전환.
    -   비용 효율성과 데이터 보안을 동시에 달성.
3.  **Agent Safety First (Antigravity Protocol):**
    -   AI 에이전트가 자율적으로 코드를 실행할 때 발생할 수 있는 사고(삭제, 유출)를 원천 차단하는 **안전 수칙(Guardrails)** 내재화.

---

## 3. KODEX 문서 체계 (Documentation Map)

이 프로젝트는 **Single Source of Truth (SSoT)** 원칙에 따라, 각 영역별 가이드라인을 모듈화하여 관리합니다.

### 🏛️ 기반 표준 (Foundation)
-   **`README.md` (This File):** 프로젝트의 대문이자 총괄 가이드.
-   **`rules.md` (Master Rule):** 에이전트 행동 강령 및 전체 규칙의 허브(Hub). **AI는 작업 전 반드시 이 파일을 먼저 읽어야 함.**
-   **`Architecture.md`**: 시스템 청사진 (FastAPI + HTMX + Hybrid LLM).
-   **`Folder_Standards.md`**: 디렉토리 구조 및 데이터 파이프라인 표준.
-   **`Code_Style.md`**: Python(Async), Type Hint, Naming Convention 등 코딩 표준.

### 🧠 AI & 도메인 가이드 (AI Engine)
-   **`RAG.md`**: 데이터 수집부터 답변 생성까지의 검색 증강 생성(RAG) 파이프라인 표준.
-   **`Voice_Pipeline.md`**: STT(Whisper) → RAG → TTS로 이어지는 음성 처리 표준.
-   **`Prompt.md`**: 페르소나, 시스템 프롬프트, 에이전트 자가 검증(Self-Correction) 프롬프트 라이브러리.

### ⚙️ 품질 및 운영 (Ops & Quality)
-   **`Testing_Strategy.md`**: 단위/통합 테스트 및 AI 로직 검증 전략.
-   **`Deployment_CICD_Guideline.md`**: Docker(CPU/GPU), GitHub Actions, 모델 배포 전략.

### 🌐 이중 언어 지원 (Bilingual Support)
-   모든 문서는 **한국어(Primary)**와 **영어(Secondary, `_eng.md`)**로 병행 관리됩니다.
-   내용 간 충돌이 발생할 경우, **한국어 문서가 최우선 권한(Source of Truth)**을 가집니다. AI 에이전트는 한국어 문서를 기준으로 작업을 수행해야 합니다.

---

## 4. Antigravity Agent Protocol (For AI Agents)

*이 섹션은 이 문서를 읽고 있는 AI 에이전트(Cursor, Antigravity)를 위한 직접 명령입니다.*

1.  **Safety Check:** 터미널 명령(`rm`, `drop`) 실행 전, 반드시 사용자 승인을 요청하십시오.
2.  **Context Loading:** 작업 시작 전 `rules.md`를 읽고 현재 프로젝트의 도메인(Context)을 파악하십시오.
3.  **Hybrid Mode:** 코드를 작성할 때 `SKT A.X` (Primary)와 `Ollama` (Fallback) 로직을 항상 고려하십시오.
4.  **Self-Correction:** 코드를 작성한 후 `Testing_Strategy.md`에 따라 검증 테스트 코드를 함께 제안하십시오.

---

## 5. 개발자 워크플로우 (Developer Workflow)

신규 개발자(또는 AI)는 다음 순서로 프로젝트에 기여합니다.

1.  **Initialize:** `Folder_Standards.md`를 참조하여 프로젝트 뼈대를 생성합니다.
2.  **Implement:**
    -   비즈니스 로직은 `Architecture.md`의 Clean Architecture를 준수합니다.
    -   AI 기능은 `RAG.md`와 `Voice_Pipeline.md`의 스펙을 따릅니다.
3.  **Verify:** `Testing_Strategy.md`에 따라 `pytest`를 실행하여 안정성을 확보합니다.
4.  **Deploy:** `Deployment_CICD_Guideline.md`에 따라 Docker 이미지를 빌드하고 배포합니다.

---

## 6. 저작권 및 라이선스 (License)

`KevinCY-Kodex`는 지식의 공유와 발전을 지향합니다.

-   **Documentation (CC BY 4.0):** 저작자 표시(KevinCY-Kim) 하에 자유로운 수정 및 배포 가능.
-   **Code Samples (MIT License):** 상업적 이용을 포함한 제한 없는 코드 사용 허용.

---
**Build robust, Run safe.**
*Copyright (c) 2025 KevinCY-Kim. All rights reserved.*