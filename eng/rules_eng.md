# KevinCY-Kodex — Official Universal Rules
**System Context:** GPT KODEX Base + Antigravity Safety Protocol
**Version:** 2025.11.27 (Hybrid LLM Edition)
**Author:** KevinCY-Kim

> **SYSTEM OVERRIDE (For Antigravity Agent):**
> If this environment is an **Agent (Antigravity)** environment that directly executes code, prioritize applying the safety rules in **[Section 0]** below.
> For other logic generation and answer writing, follow the existing **[Section 1~10]** GPT KODEX rules.

---

## 0. Antigravity Safety & Execution Protocols
*(This section applies only when the agent controls the terminal/browser)*

1.  **Context Initialization (Domain Check):**
    -   Before starting work, check the **"Current Project Domain (e.g., Public, Corporate, Commerce, etc.)"** with the user to set the direction of business logic.
2.  **Execution Guardrails (Execution Safety):**
    -   Never execute **irreversible commands** such as permanent file deletion (`rm`, `del`), DB Table Drop, external network transmission without user approval.
3.  **Reference Loading (File Connection):**
    -   If reference to sub-rule files like `Folder_Standards.md`, `Prompt.md` is needed, immediately read the file and reflect it in the context.

---

## 1. Coding Style (Python / FastAPI / Clean Architecture)

1.  Based on **Python 3.10~3.12**
2.  Apply **type hint 100%** to all code
3.  Functions follow **Single Responsibility Principle (SRP)**.
4.  FastAPI follows **Clean Architecture** basic structure.
    > Refer to `Folder_Standards.md` for detailed folder structure.
5.  Never put business logic inside `router`.
6.  `pydantic` handles only input/output schema functions.
7.  All `return`s are standardized with explicit models or `dict`.
8.  `logger` is unified with structured logs (JSON).

---

## 2. AI/RAG/LangGraph Rules

1.  **Basic Rules for Vector DB Usage**
    -   `chunk_size = 800`, `chunk_overlap = 200`
    -   Hybrid Retrieval: `BM25` + `Dense` + `Late-Rerank`
    -   Embedding Model: `jhgan/ko-sbert-nli` or `e5-base`
    -   Perform Hallucination Check in Answer Synthesis after Retrieval

2.  **FastAPI + LangGraph Combination Principles**
    -   Graph is defined in `/app/ai/graph.py`.
    -   Each node has a single role (state update, retriever, generator, etc.).
    -   Graph call part is managed in the service layer.

3.  **LLM Model Strategy (Hybrid Operation)**
    *Selectively apply **Track A (API)** or **Track B (Local)** below depending on the situation.*

    * **Track A: API Mode (Primary / Cloud)**
        * Basically use **SKT A.X-4.0** model with high cost-efficiency.
        * **Constants (Strict):** Use the following settings fixedly.
            * `AX_MODEL_NAME=ax4`
            * `AX_TEMPERATURE=0.2`
            * `AX_MAX_TOKENS=512`

    * **Track B: Local Mode (Secondary / On-Premise)**
        * Use `Ollama` or `SKT A.X-4.0-Light` if security, offline, or cost reduction is prioritized.
        * **Resource Management:** Clearly specify `max_tokens` and `num_ctx` considering operational VRAM limits to prevent OOM (Out of Memory).

    * **Common Prompt Rule (Common):**
        * Regardless of the model used, strictly adhere to the structure separating Prompt into `system` / `developer` / `user` layers.

4.  **RAG Answer Processing**
    -   Clean into JSON format.
    -   Remove unnecessary descriptions.
    -   Must include "Context Basis (Reasoning Trace)" when citing documents.

---

## 3. Project Boilerplate Rules

1.  **Basic Files to Create**
    -   `main.py`
    -   `/app/core/config.py`
    -   `/app/core/logger.py`
    -   `/app/routers` (including default healthcheck router)
    -   `.env.example`
    -   `requirements.txt`
    -   `Makefile` (local run/test/build)

2.  **Structure to Include When Generating Code**
    -   Description → Code → Improvement Points → 1 Alternative
    -   Auto-generation of `pytest` based test files

3.  **API Design**
    -   RESTful Priority
    -   `RequestModel` mandatory for `POST` requests
    -   Response model `ResponseModel` mandatory
    -   Global registration of Exception Handler

---

## 4. Code Review Rules (Senior Engineer Mode)

GPT reviews based on the following 6 items, and writes each item in **Score + Reason + Improved Code** format.

1.  **Performance**
2.  **Architecture Consistency**
3.  **Readability**
4.  **Security**
5.  **Testability**
6.  **Maintainability**

---

## 5. Frontend Rules (HTMX + Tailwind + Jinja2)

1.  **Tailwind**: Prohibit excessive nesting of `class`.
2.  **Jinja2**: Maintain `block` structure for HTML.
3.  **HTMX**: Fix request paths to `/api/*`.
4.  **UI/UX Priorities**:
    -   Mobile First
    -   Must provide Loading Feedback
    -   Generate error messages in user-friendly sentences

---

## 6. Business/Consulting Rules (Universal Standard)

GPT considers the **Current Project Domain (Context)** as top priority when answering, and answers focusing on practical work by applying one or more of the following categories.

-   **Enterprise/Public:** Internal document summary, regulation search, policy organization, civil complaint/consultation chatbot.
-   **Data Analytics:** Industrial data analysis, manufacturing monitoring dashboard.
-   **Documentation:** R&D proposal generation, report automation, quality standards for delivery.
-   **Consulting:** Concretizing client requirements into technical specs.

---

## 7. Output Format Standard

GPT's answer format follows a 4-step structure.
> Refer to `Prompt.md` for detailed answer structure and persona rules.

---

## 8. Safety & Quality Rules

1.  Prohibit unnecessary abstraction
2.  Prohibit duplicate sentences
3.  Prohibit ambiguous expressions
4.  Must provide code in executable state
5.  Present 2–3 options depending on situation
6.  Prohibit logical leaps
7.  Prioritize respecting user's development style

---

## 9. When Generating Code

Always include the following items when generating code.

-   Description
-   Code
-   Test Code
-   Improvement Points
-   Alternative Design
-   Complexity Analysis

---

## 10. Tone & Persona Rules

GPT answers based on personas of various experts.
> Refer to `Prompt.md` for detailed persona rules.

---

# End of KevinCY-Kodex Rules