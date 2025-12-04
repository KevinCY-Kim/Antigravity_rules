# KevinCY-Kodex — Voice Pipeline Guideline
**Version:** 2025.11.27 (Universal Standard)
**Scope:** STT · TTS · RAG · LangGraph · Voice Chatbot

---

## 1. Overview

The voice-based Q&A pipeline consists of the following 4 steps:

1.  **Audio Input**: Voice file upload
2.  **STT (Speech-to-Text)**: Convert speech to text
3.  **Answer Generation**: RAG or Hybrid LLM based answer generation
4.  **TTS (Text-to-Speech)**: Convert text to speech

This entire flow is provided as a single `Service` function in `FastAPI`, and the `Router` is responsible only for file reception and response return.

---

## 2. File Structure

The recommended folder structure is as follows.
Voice processing related files are placed according to the standard folder structure of `KevinCY-Kodex`.

-   **STT/TTS Modules**: `/app/ai/stt/`, `/app/ai/tts/`
    -   Located in `ai` directory as part of AI tech stack.
-   **Voice Pipeline Workflow**: `/app/ai/voice_pipeline.py`
    -   Located in `ai` directory as it is an AI workflow connecting STT, RAG, TTS.
-   **Voice Service**: `/app/services/voice_service.py`
    -   Located in `services` directory as it is business logic connecting Router and AI pipeline.

> Refer to `Folder_Standards.md` for detailed overall folder structure.

---

## 3. Audio Input Rules

### 3.1 Upload Rules
-   User voice is received in `UploadFile` format.
-   Allowed extensions: `wav`, `mp3`, `m4a`, `ogg`
-   File size limit (e.g., 5~10MB) can be applied.
-   File save path is generated in the form of `/tmp/{user_id}_{timestamp}.{ext}`.

### 3.2 Security Rules (Agent Safety)
-   **Cleanup:** Uploaded files must be **automatically deleted (unlink)** after processing is complete (`finally` block) to prevent storage leaks.
-   Do not use uploaded filenames as system paths directly (UUID conversion).
-   Process only verified audio files after checking `MIME` type.

---

## 4. STT (Speech-to-Text) Specification

### 4.1 Engine Selection
-   **Recommended Models**:
    -   `Faster-Whisper` (If GPU available - Primary)
    -   `Whisper Small/Medium` (CPU or limited GPU environment - Secondary)

### 4.2 STT Option Rules
-   `language`: `auto` or `ko` (Korean priority)
-   `noise_suppression`: `on` (Apply depending on environment)
-   `beam_size`: 1~3 (Balance between speed and accuracy)
-   `vad_filter`: Activate if needed (Remove silence sections)

### 4.3 STT Output Format
-   `transcribed_text`: String
-   `confidence` (optional): Reliability
-   `segments` (optional): Time segment info

### 4.4 STT Quality Correction Rules
-   Correct continuous words/truncation phenomena.
-   Remove unnecessary noise (Filler words) like "Um, well...".
-   Pronouns (this/that/he/she) can be reconstructed to fit the question context.

---

## 5. Query Reconstruction (Optional)

Since voice often contains unclear sentences, the following reconstruction rules can be applied:

-   Automatic conversion to natural question form
-   Context-based supplementation of missing words
-   Division into TTS-friendly sentence lengths
-   **Example**: "Where get birth grant?" → "I am curious about the application location and procedure for Paju City birth grant."

---

## 6. RAG / LLM Answer Generation

### 6.1 Selection Criteria (Hybrid LLM Strategy)
*Follow settings in Master Rule and Code_Style.md.*

-   **RAG Priority (Primary)**: Use `SKT A.X` (API) model to perform accurate reasoning and sentence generation.
-   **Fallback (Secondary)**: Use `Ollama` for simple small talk or in local environments.

### 6.2 RAG Criteria
-   Apply `Chunk` retrieval and `Rerank`
-   Prioritize document-based reliability (Suppress Hallucination)
-   Must include evidence sentences (Reasoning Trace)

### 6.3 Answer Output Form
-   `text_answer`: Refined text for TTS
-   `source_chunks` (optional): Source info
-   `structured_json` (optional): Structured data
-   `followup_questions` (optional): Suggest follow-up questions

---

## 7. TTS (Text-to-Speech)

### 7.1 Engine
-   `gTTS` (For simple MVP/PoC)
-   Or expandable to commercial/open-source TTS engines (Edge-TTS, etc.)

### 7.2 TTS Rules
-   Cut text naturally in units of 120~200 characters.
-   Consider elderly/beginner friendly options (`slow` or `0.9x speed`).
-   Consider sound quality enhancement options (Apply max bit rate).

### 7.3 File Output
-   **Path**: `/static/tts/{user_id}_{timestamp}.mp3`
-   **Return**: URL format (`/static/tts/...`)

### 7.4 TTS File Management
-   **TTL (Time-To-Live):** Apply a scheduler where generated files are deleted after a certain period (e.g., 1 hour ~ 24 hours).
-   Implement exception handling for `mp3`/`wav` conversion errors.

---

## 8. FastAPI Router Rules

### 8.1 Router Role
-   File reception and temporary storage
-   Service call
-   Return service result (JSON + TTS URL)

### 8.2 Router Response Format
```json
{
  "query": "User Question",
  "answer": "Generated Answer",
  "tts_url": "/static/tts/xxx.mp3",
  "sources": [...]
}
```

---

## 9. Service Layer Workflow

### 9.1 Execution Flow
1.  **STT**: Convert audio file to text
2.  **Refine**: Text refinement and question reconstruction (Query Reconstruction)
3.  **Generation**: **LangGraph RAG (Hybrid LLM)** based answer generation
    * *Comply with Master Rule's SKT A.X(API) / Ollama(Local) strategy*
4.  **Post-processing**: Text post-processing (TTS friendly conversion)
5.  **TTS**: Convert text to voice file
6.  **Return**: Return JSON response and audio file URL

### 9.2 Exception Handling
-   **STT Failure**: "Voice is not clear. Please speak again." (Voice guide)
-   **RAG/LLM Failure**: "Could not find relevant information." (Default Fallback message)
-   **TTS Failure**: Return text only (Induce client-side reading processing)

---

## 10. Post-Processing Rules

### 10.1 Text Cleaning (Voice UX)
-   Do not make sentences too long (Consider TTS breathing).
-   Induce `pause` by appropriately adding commas (`,`) and periods (`.`) so the flow is natural during voice output.
-   Maintain a consistent **polite and stable tone (Formal/Polite)** for public/office service answers.

### 10.2 Domain Specific Action (Universal)
-   **Context Check:** If information irrelevant to the current project domain (e.g., Paju City, Internal Regulations) is detected (other local governments, etc.), filter it or add a warning comment.
-   **Disclaimer:** Automatically add the phrase "Please reconfirm latest information at official site/department in charge" for volatile systems or regulations.

---

## 11. Logging Rules

-   **Mandatory Log Items**: `user_id`, `STT_latency`, `RAG_latency`, `TTS_latency`, `Total_latency`, `Error_Code`
-   **Format**: Recommend saving logs in **JSON structure** for easy analysis.
-   **Privacy**: Be careful not to leave voice data itself or PII (Personal Information) in logs.

---

## 12. Quality Evaluation

-   **STT Quality Metrics**: `WER` (Word Error Rate), Recognition accuracy vs utterance length
-   **RAG Quality Metrics**: `Recall@K` (Search recall), `Faithfulness` (Evidence fidelity)
-   **User Experience Metrics**: `End-to-End Latency` (Response speed), Voice playback success rate, User re-question rate

---

## 13. Expansion Strategy (Future Roadmap)

-   **GPU Acceleration:** `Faster-Whisper` + CUDA optimization
-   **High-Quality TTS:** Replace with commercial `Neural TTS` (ElevenLabs, OpenAI)
-   **Streaming:** Implement `WebRTC` based real-time voice input and TTS streaming response
-   **Emotion:** Emotion/Tone adjustment model integration (Warm tone, Business tone, etc.)

End of KevinCY-Kodex Voice Pipeline Guideline