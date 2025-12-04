# KevinCY-Kodex — Folder Standards
**Version:** 2025.11.27 (Universal Standard)
**Scope:** FastAPI · LangGraph · RAG · STT/TTS · Web(HTMX+Tailwind)
**Viewer Note:** Please render tree structures in fixed-width font (Code Block).
---

## 1. Overview

This document defines the standard rules for the project file and folder structure.
The project is composed of the following core layers according to Clean Architecture principles:

-   `core`: Core settings, logger, security, etc.
-   `routers`: API endpoints
-   `services`: Business logic
-   `repositories`: IO such as database, file, external API
-   `models`: Pydantic schemas
-   `ai`: AI engines such as LLM, RAG, LangGraph
-   `utils`: Shared utilities
-   `templates` & `static`: Web frontend

Each layer must possess **Single Responsibility Principle (SRP)** and adhere to the direction of dependency between layers.

---

## 2. Root Folder Structure (Top-Level)

Below is the top-level structure of the `KevinCY-Kodex` project.

```
project-root/
├── app/                  # Application Core Logic
│   ├── core/             # Settings, Security, Logger
│   ├── routers/          # API Endpoints (Controller)
│   ├── services/         # Business Logic
│   ├── repositories/     # I/O Layer (DB, File, API)
│   ├── models/           # Pydantic Schemas
│   ├── ai/               # AI Engine (LLM, RAG, Graph)
│   └── utils/            # Shared Utilities
│
├── templates/            # Jinja2 HTML Templates
│   ├── layout/           # Base Layouts
│   ├── pages/            # Page Content
│   └── partials/         # HTMX Components
│
├── static/               # Static Assets (CSS, JS, Images)
├── tests/                # Pytest Cases
├── data/                 # (Optional) Local Data Storage
│
├── .env                  # Environment Variables (Git Ignored)
├── .env.example          # Env Template
├── requirements.txt      # Python Dependencies
├── README.md             # Project Documentation
├── Makefile              # Automation Scripts
└── main.py               # App Entry Point
```

---

## 3. /app/core — Core Settings Layer

Manages base system components such as core settings, logger, and exception handling.

-   **Included Files**: `config.py`, `logger.py`, `exception_handlers.py`, `security.py`, etc.
-   **Responsibilities**:
    -   Loading environment variables
    -   Common logger settings
    -   Security/Authentication policies
    -   FastAPI global exception handling

---

## 4. /app/routers — API Endpoint Layer

Defines FastAPI endpoints. Never includes business logic.

-   **Rules**:
    -   Filename: `xxx_router.py`
    -   Router variable name: `router`
    -   Role: Receive `Request` → Call `Service` → Return `Response`
-   **Examples**: `chat_router.py`, `voice_router.py`, `health_router.py`

---

## 5. /app/services — Business Logic Layer

Responsible for combining all business logic and workflows.

-   **Rules**:
    -   Filename: `xxx_service.py`
    -   Coordinator role between `Router` and `Repository`/`AI` layers
    -   Delegates external system call logic to `Repository`
-   **Examples**: `chat_service.py`, `voice_service.py`, `rag_service.py`

---

## 6. /app/repositories — IO (DB/File/API) Layer

Handles all IO such as DB, file system, external API, Vector DB.

-   **Rules**:
    -   Filename: `xxx_repository.py`
    -   Maintain pure function form as much as possible
    -   Prohibit inclusion of business logic
-   **Examples**: `document_repository.py`, `vector_repository.py`, `user_repository.py`

---

## 7. /app/models — Pydantic Models & DTOs

Defines Pydantic models used for `Request`/`Response`/`DTO`.

-   **Rules**:
    -   Filename: `xxx_model.py`
    -   Prohibit inclusion of business logic
-   **Examples**: `chat_model.py`, `voice_model.py`, `rag_model.py`

---

## 8. /app/ai — LLM, RAG, LangGraph Engine Layer

Area responsible for all AI-related components.

-   **Recommended Detailed Structure**:
    ```
    /app/ai/
    ├── graph.py              # LangGraph Main Workflow
    ├── llm_client.py         # LLM Client Wrapper (API/Local)
    ├── ingest/               # Document Loading Logic
    ├── chunk/                # Text Splitter Logic
    ├── embed/                # Embedding Model Logic
    ├── retriever/            # Vector DB Search Logic
    ├── reranker/             # Re-ranking Logic
    ├── generator/            # Prompt & Generation Logic
    ├── stt/                  # Speech-to-Text Logic
    └── tts/                  # Text-to-Speech Logic
    ```
-   **Roles**: LangGraph pipeline, LLM calls, RAG components, Voice processing (STT/TTS)

---

## 9. /app/utils — Common Utility Functions

Responsible for small pure functions, common logic, format conversion, string processing, etc.

-   **Examples**: `string_utils.py`, `time_utils.py`, `file_utils.py`

---

## 10. /templates — Jinja2 HTML Templates

Manages UI templates based on HTMX and Tailwind.

-   **Structure Standard**:
    ```
    /templates/
    ├── layout/               # base.html, header.html, footer.html
    ├── pages/                # index.html, chat.html, voice_chat.html
    └── partials/             # chat_message.html, loader.html
    ```

---

## 11. /static — CSS, JS, Assets

Stores static resources such as compiled CSS, JS, images, audio files.

-   **Configuration Example**:
    ```
    /static/
    ├── css/                  # tailwind.css (Compiled)
    ├── js/                   # main.js (HTMX Extensions)
    ├── img/                  # Images
    └── tts/                  # Generated Audio Files (Temp)
    ```

---

## 12. /tests — Test Directory

Stores `pytest`-based test codes.

-   **Rules**:
    -   Filename: `test_xxx.py`
    -   Can be configured by separating unit tests and integration tests

---

## 13. Top-Level Files

-   `.env` / `.env.example`: Environment variable files (API Key, DB URL, etc.)
-   `requirements.txt`: List of Python dependency packages
-   `Makefile`: Execution/Test/Deployment scripts
-   `README.md`: Project overview and guide
-   `main.py`: FastAPI app execution entry point (uvicorn.run)

---

## 14. Dependency Rules Between Layers

Dependencies must always point from outside to inside.

-   **Allowed**:
    -   `routers` → `services`
    -   `services` → `repositories` / `ai`
-   **Globally Available**: `core`, `utils`, `models`
-   **Prohibited**: Reverse dependencies (e.g., `repositories` → `services`)

---

## 15. Project Structure for Expansion (Large Scale)

For large-scale projects, it is recommended to separate modules by functional domain.

-   **Example**:
    ```
    /app/
      ├── domain_chat/
      │   ├── chat_router.py
      │   ├── chat_service.py
      │   └── chat_model.py
      │
      ├── domain_voice/
      │   ├── voice_router.py
      │   ├── voice_service.py
      │   └── voice_model.py
      │
      └── ...
    ```

---

## 16. Data Pipeline Structure (Universal)

Standard data folder for RAG/Document processing projects can include a `data/` folder to manage raw data and processed data.

```
project-root/
  ├── data/
  │   ├── raw/        # Raw PDF/Documents
  │   ├── cleaned/    # Preprocessed Text
  │   ├── chunks/     # Chunk Storage
  │   └── embeddings/ # Vector Index
  └── ...
```

---

## 17. Folder Management Principles in Operations/Deployment

-   All environments (`local`, `staging`, `prod`) maintain the same folder structure.
-   `data`, `embedding`, `TTS` files, etc., are managed by applying version rules.
-   Old temporary files, logs, and TTS voice files are cleaned up periodically.
-   `.env` files are never included in the version control system (Git).

---

End of KevinCY-Kodex Folder Standards