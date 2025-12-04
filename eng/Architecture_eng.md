# KevinCY-Kodex — Architecture Guideline
**Version:** 2025.11.27 (Universal Standard)
**Scope:** FastAPI + LangGraph + RAG + STT/TTS + Web Frontend
**Reference:** This document is a detailed technical sub-document of `KevinCY-Kodex (Master Rule)`.

---

## 1. System Overview

This system aims for a 'Hybrid AI Service' and consists of the following subsystems:

-   **API Backend**: FastAPI (Clean Architecture)
-   **AI Engine**: LangGraph Orchestration + Hybrid LLM (SKT A.X / Ollama)
-   **Voice Pipeline**: STT (Whisper) + TTS (Generative)
-   **Web Frontend**: HTMX + Tailwind + Jinja2 (SSR/SPA Hybrid)
-   **Infra & Ops**: Docker, Conda, Local/Prod Environment Separation

---

## 2. Backend Layered Architecture

### 2.1 Directory Standards and Layer Responsibilities
The project's folder structure and physical placement of each layer follow the **`Folder_Standards.md`** document.
*(Must adhere to SRP principles and Clean Architecture defined in Master Rule)*

---

## 3. Request Lifecycle

### 3.1 Text-Based Query (RAG / Chatbot)

1.  **Client**: `POST` request to `/api/chat`
2.  **Router**: Calls `Service` after RequestModel validation
3.  **Service**:
    -   Retrieves User Context (Metadata)
    -   Executes **LangGraph Pipeline** (Retrieval → Reasoning → Generation)
    -   Maps result to `ResponseModel`
4.  **Router**: Returns final JSON response
5.  **Logger**: Records Latency and Token Usage (JSON structure)

### 3.2 Voice-Based Query (STT → RAG → TTS)

1.  **Client**: Uploads Audio Blob to `/api/voice-chat`
2.  **Router**: Calls `Service` after saving temporary file (Temp)
3.  **Service**:
    -   **STT Engine**: Audio → Text conversion
    -   **Text Pipeline**: Reuses RAG flow from 3.1 (Code Reusability)
    -   **TTS Engine**: Generated Answer (Text) → Audio conversion
4.  **Router**: Returns Text Answer (JSON) + Audio File URL
5.  **Cleanup**: Deletes temporary upload file after request completion (Complies with Antigravity Safety Rule)

---

## 4. AI / RAG / LangGraph Architecture

> **Note:** Detailed implementation figures (Chunk size, etc.) prioritize settings in `Master Rule`.

### 4.1 RAG Pipeline (Standard Flow)

#### Ingestion
-   Source: Unstructured data like PDF, DOCX, HWP
-   Process: Text Extraction → Metadata Tagging (Source, Page, Date)

#### Chunking & Embedding
-   **Spec:** `chunk_size = 800`, `chunk_overlap = 200` (Sync with Master Rule)
-   **Model:** `ko-sbert-nli` (Optimized for Semantic Search)
-   **DB:** Vector DB (FAISS/Chroma) + Metadata Filtering

#### Retrieval (Hybrid)
-   **Algorithm:** `BM25` (Keyword) + `Dense` (Semantic)
-   **Rerank:** Re-ranking with Cross-Encoder after candidate extraction (Late Reranking)

#### Answer Generation
-   **Model Strategy:**
    -   **Default:** `SKT A.X-4.0` (API)
    -   **Fallback:** `Ollama` (Local)
-   **Format:** Specify Reasoning Trace and Citation when generating answers

### 4.2 LangGraph Structure
-   **Location:** `/app/ai/graph.py`
-   **Node Design:**
    -   `router_node`: Question Intent Classification (General vs RAG)
    -   `retrieval_node`: Document Search
    -   `generation_node`: Answer Generation
    -   `grade_node`: Answer Quality Evaluation (Hallucination Check)

---

## 5. Voice Pipeline (STT / TTS)

### 5.1 STT (Speech-to-Text)
-   **Engine:** `Faster-Whisper` (Local/GPU Optimized)
-   **Optimization:** `Beam Size=5`, `VAD_filter=True`
-   **Language:** Korean (ko) Auto Detection

### 5.2 TTS (Text-to-Speech)
-   **Engine:** `gTTS` (Basic) or Commercial API
-   **Output:** `.mp3` or `.wav`
-   **Naming:** `{timestamp}_{uuid}.mp3` (Conflict Prevention)

### 5.3 Pipeline Rules
-   STT and TTS are managed as separate modules in the `Service` layer.
-   Router handles only File I/O without business logic.

---

## 6. Frontend Architecture

### 6.1 Template Structure

```
/templates/
  ├── layout/
  │   ├── base.html
  │   ├── header.html
  │   └── footer.html
  ├── pages/
  │   ├── index.html
  │   ├── chat.html
  │   └── voice_chat.html
  └── partials/
      ├── chat_message.html
      └── loader.html
```

### 6.2 HTMX Integration Rules
-   **Namespace:** All `hx-post`, `hx-get` requests use the `/api/` prefix.
-   **Implementation Pattern:**
    -   **Form Example:** `<form hx-post="/api/chat" hx-target="#chat-window" hx-swap="beforeend">`
    -   **Loading:** Must display Spinner/Loading Bar during AI processing using `hx-indicator`.

### 6.3 UI/UX Design Principles
-   **Mobile First:** Design for mobile screens first using Tailwind's `md:`, `lg:` prefixes.
-   **Card UI:** Compose content with **Card-type UI** with sufficient whitespace to improve readability.
-   **Feedback:** When 4xx/5xx errors occur, provide Toast or Modal notifications with **user-friendly sentences** instead of exposing system logs.

---

## 7. Infra / Environment / Deployment

### 7.1 Environment Separation and Configuration
-   **Environment:** Clearly separate `local`, `staging`, and `prod` environments.
-   **Config:** Manage settings for each environment with `.env` files, including mandatory variables from Master Rule like `AX_MODEL_NAME`.

### 7.2 Dependency & Resource Management
-   **Package:** Fix dependencies and Python versions via `conda` environment, `requirements.txt`, or `env.yml`.
-   **Isolation:** Engines requiring GPU resources like STT/LLM are separated from the web server at the process/container level to ensure operational stability.

### 7.3 Logging & Monitoring
-   **Metrics:** Monitor Request Count (RPS), Response Time (Latency), and Error Rate as key indicators.
-   **Log Fields:** All logs must include `path`, `method`, `status_code`, `latency`, and `user_id` for traceability.

---

## 8. Cross-Cutting Concerns

### 8.1 Global Error Handling
-   **Location:** Global exception handler is defined in `/app/core/exception_handlers.py`.
-   **Response Format (Standard):**
    ```json
    {
      "error_code": "E400",
      "message": "Input value is invalid.",
      "detail": "Field 'query' is missing."
    }
    ```

### 8.2 Security
-   **CORS:** Apply a whitelist policy allowing only minimal Origins.
-   **Privacy:** Sensitive information like PII or API Keys must be masked when saving logs.

### 8.3 Configuration Management
-   **Library:** Load settings in a Type-safe manner using `Pydantic`'s `BaseSettings` in `/app/core/config.py`.

---

## 9. Architectural Manifesto (Core Principles)
*Agents should use the following 5 principles as the final checklist when designing code.*

1.  **Clean Architecture First:** All code prioritizes adherence to layer dependency rules. (Controller → Service → Repository/AI)
2.  **Thin Router, Fat Service:** `Router` only handles request validation and return, while all business logic is concentrated in the **`Service`** layer.
3.  **Decoupled AI Engine:** AI/LLM parts (LangGraph, Prompt) are separated from business logic so that service code is not affected even if models change.
4.  **Universal Extensibility:** Aim for a module structure that is not dependent on a specific domain (e.g., Paju City) and can be immediately extended/replicated to other domains.
5.  **Reusable Pipeline:** STT, TTS, and RAG pipelines are implemented as reusable **Class or Module** units, not disposable functions.

---

# End of Architecture Guideline