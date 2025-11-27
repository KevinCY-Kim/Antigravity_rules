# KevinCY-Kodex — Voice Pipeline Guideline
**Version:** 2025.11.27 (Universal Standard)
**Scope:** STT · TTS · RAG · LangGraph · Voice Chatbot

---

## 1. Overview

음성 기반 질의응답 파이프라인은 다음 4단계로 구성됩니다.

1.  **Audio Input**: 음성 파일 업로드
2.  **STT (Speech-to-Text)**: 음성을 텍스트로 변환
3.  **Answer Generation**: RAG 또는 Hybrid LLM 기반 답변 생성
4.  **TTS (Text-to-Speech)**: 텍스트를 음성으로 변환

이 흐름 전체를 `FastAPI`의 단일 `Service` 함수로 묶어 제공하며, `Router`에서는 파일 수신과 응답 반환만 담당합니다.

---

## 2. File Structure

권장 폴더 구조는 다음과 같습니다.
음성 처리 관련 파일들은 `KevinCY-Kodex`의 표준 폴더 구조에 따라 배치됩니다.

-   **STT/TTS 모듈**: `/app/ai/stt/`, `/app/ai/tts/`
    -   AI 기술 스택의 일부로, `ai` 디렉토리 내에 위치합니다.
-   **음성 파이프라인 워크플로우**: `/app/ai/voice_pipeline.py`
    -   STT, RAG, TTS를 연결하는 AI 워크플로우이므로 `ai` 디렉토리에 위치합니다.
-   **음성 서비스**: `/app/services/voice_service.py`
    -   라우터와 AI 파이프라인을 연결하는 비즈니스 로직이므로 `services` 디렉토리에 위치합니다.

> 자세한 전체 폴더 구조는 `Folder_Standards.md` 문서를 참조하세요.

---

## 3. Audio Input Rules

### 3.1 업로드 규칙
-   사용자 음성은 `UploadFile` 형태로 수신합니다.
-   허용 확장자: `wav`, `mp3`, `m4a`, `ogg`
-   파일 크기 제한(예: 5~10MB)을 적용할 수 있습니다.
-   파일 저장 경로는 `/tmp/{user_id}_{timestamp}.{ext}` 형태로 생성합니다.

### 3.2 보안 규칙 (Agent Safety)
-   **Cleanup:** 업로드된 파일은 처리 완료 후(`finally` 블록) 반드시 **자동 삭제(unlink)** 하여 스토리지 누수를 방지합니다.
-   업로드된 파일명을 그대로 시스템 경로로 사용하지 않습니다(UUID 변환).
-   `MIME` 타입 확인 후 검증된 오디오 파일만 처리합니다.

---

## 4. STT (Speech-to-Text) Specification

### 4.1 엔진 선택
-   **권장 모델**:
    -   `Faster-Whisper` (GPU 사용 가능한 경우 - Primary)
    -   `Whisper Small/Medium` (CPU 또는 제한된 GPU 환경 - Secondary)

### 4.2 STT 옵션 규칙
-   `language`: `auto` 또는 `ko` (한국어 우선)
-   `noise_suppression`: `on` (환경에 따라 적용)
-   `beam_size`: 1~3 (속도와 정확도 균형)
-   `vad_filter`: 필요 시 활성화 (무음 구간 제거)

### 4.3 STT 출력 포맷
-   `transcribed_text`: 문자열
-   `confidence`(optional): 신뢰도
-   `segments`(optional): 시간 구간 정보

### 4.4 STT 품질 보정 규칙
-   연속 단어/짤림 현상을 보정합니다.
-   "음, 그..." 등 불필요한 잡음(Filler words)을 제거합니다.
-   대명사(이거/저거/그분)를 질문 문맥에 맞게 재구성할 수 있습니다.

---

## 5. Query Reconstruction (Optional)

음성은 불명확한 문장이 많기 때문에 다음과 같은 재구성 규칙을 적용할 수 있습니다.

-   자연스러운 질문 형태로 자동 변환
-   누락된 단어를 문맥 기반으로 보완
-   TTS 친화적인 문장 길이로 분할
-   **예시**: "출산지원금 어디서 받지?" → "파주시 출산지원금 신청 장소와 절차가 궁금합니다."

---

## 6. RAG / LLM Answer Generation

### 6.1 선택 기준 (Hybrid LLM Strategy)
*Master Rule 및 Code_Style.md의 설정을 따릅니다.*

-   **RAG 우선 (Primary)**: `SKT A.X` (API) 모델을 사용하여 정확한 추론과 문장 생성을 수행합니다.
-   **Fallback (Secondary)**: 단순 잡담이나 로컬 환경에서는 `Ollama`를 사용합니다.

### 6.2 RAG 기준
-   `Chunk` retrieval 및 `Rerank` 적용
-   문서 기반 신뢰도 우선 (Hallucination 억제)
-   근거 문장(Reasoning Trace)을 반드시 포함

### 6.3 답변 출력 형태
-   `text_answer`: TTS용 정제 텍스트
-   `source_chunks`(optional): 출처 정보
-   `structured_json`(optional): 구조화 데이터
-   `followup_questions`(optional): 후속 질문 제안

---

## 7. TTS (Text-to-Speech)

### 7.1 엔진
-   `gTTS` (간단한 MVP/PoC 용)
-   또는 상용/오픈소스 TTS 엔진으로 확장 가능 (Edge-TTS 등)

### 7.2 TTS 규칙
-   텍스트는 120~200자 단위로 자연스럽게 끊습니다.
-   어르신/초보자 친화 옵션(`slow` or `0.9x speed`)을 고려합니다.
-   음질 강화 옵션(최대 bit rate 적용)을 고려합니다.

### 7.3 파일 출력
-   **경로**: `/static/tts/{user_id}_{timestamp}.mp3`
-   **반환**: URL 형태 (`/static/tts/...`)

### 7.4 TTS 파일 관리
-   **TTL(Time-To-Live):** 생성된 파일은 일정 기간(예: 1시간~24시간) 후 삭제되는 스케줄러를 적용합니다.
-   `mp3`/`wav` 변환 오류에 대비한 예외 처리를 구현합니다.

---

## 8. FastAPI Router Rules

### 8.1 Router 역할
-   파일 수신 및 임시 저장
-   서비스 호출
-   서비스 결과(JSON + TTS URL) 반환

### 8.2 Router 응답 형식
```json
{
  "query": "사용자 질문",
  "answer": "생성된 답변",
  "tts_url": "/static/tts/xxx.mp3",
  "sources": [...]
}
```

---

## 9. Service Layer Workflow

### 9.1 전체 흐름 (Execution Flow)
1.  **STT**: 오디오 파일을 텍스트로 변환
2.  **Refine**: 텍스트 정제 및 질문 재구성 (Query Reconstruction)
3.  **Generation**: **LangGraph RAG (Hybrid LLM)** 기반 답변 생성
    * *Master Rule의 SKT A.X(API) / Ollama(Local) 전략 준수*
4.  **Post-processing**: 텍스트 후처리 (TTS 친화적 변환)
5.  **TTS**: 텍스트를 음성 파일로 변환
6.  **Return**: JSON 응답 및 오디오 파일 URL 반환

### 9.2 예외 상황 처리
-   **STT 실패**: "음성이 명확하지 않습니다. 다시 말씀해 주세요." (음성 안내)
-   **RAG/LLM 실패**: "관련 정보를 찾을 수 없습니다." (기본 Fallback 메시지)
-   **TTS 실패**: 텍스트만 반환 (Client 측 읽기 처리 유도)

---

## 10. Post-Processing Rules

### 10.1 텍스트 정제 (Voice UX)
-   문장을 너무 길게 만들지 않습니다 (TTS 호흡 고려).
-   음성 출력 시 흐름이 자연스럽도록 쉼표(`,`)와 마침표(`.`)를 적절히 추가하여 `pause`를 유도합니다.
-   공공/오피스 서비스 답변은 **정중하고 안정적인 톤(하십시오체/해요체)**을 일관되게 유지합니다.

### 10.2 Domain Specific Action (Universal)
-   **Context Check:** 현재 프로젝트 도메인(예: 파주시, 사내규정)과 무관한 정보(타 지자체 등)가 감지되면 필터링하거나 경고 멘트를 추가합니다.
-   **Disclaimer:** 변동성 있는 제도나 법규는 "최신 정보는 공식 사이트/담당 부서에서 재확인 바랍니다"라는 문구를 자동으로 추가합니다.

---

## 11. Logging Rules

-   **필수 로그 항목**: `user_id`, `STT_latency`, `RAG_latency`, `TTS_latency`, `Total_latency`, `Error_Code`
-   **Format:** 로그는 분석이 용이하도록 **JSON 구조**로 저장하는 것을 권장합니다.
-   **Privacy:** 음성 데이터 자체나 PII(개인정보)는 로그에 남기지 않도록 주의합니다.

---

## 12. Quality Evaluation

-   **STT 품질 지표**: `WER` (Word Error Rate), 발화 길이 대비 인식 정확도
-   **RAG 품질 지표**: `Recall@K` (검색 재현율), `Faithfulness` (근거 충실도)
-   **사용자 경험 지표**: `End-to-End Latency` (응답 속도), 음성 재생 성공률, 사용자 재질문 비율

---

## 13. 확장 전략 (Future Roadmap)

-   **GPU Acceleration:** `Faster-Whisper` + CUDA 최적화
-   **High-Quality TTS:** 상업용 `Neural TTS` (ElevenLabs, OpenAI)로 교체
-   **Streaming:** `WebRTC` 기반 실시간 음성 입력 및 TTS 스트리밍 응답 구현
-   **Emotion:** 감정/톤 조절 모델 연동 (따뜻한 톤, 사무적 톤 등)

End of KevinCY-Kodex Voice Pipeline Guideline