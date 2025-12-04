# KevinCY-Antigravity: AI Engineering Standard & Playbook

**Version:** 2025.11.27 (Universal Hybrid Edition)
**Author:** KevinCY-Kim (System Architect)
**Compatible With:** GPT KODEX, Google Antigravity

---

## 1. Introduction

`KevinCY-Antigravity` is a **Unified Engineering Standard** designed to build scalable AI applications (RAG, Voice Bot).
Based on **Clean Architecture**, **Hybrid AI Strategy**, and **Agent Safety**, it provides a playbook that systematizes the development process from planning to deployment.

This document serves as a **Root Instruction** for not only the **Development Team** but also the **AI Agent (Antigravity)** to understand the project context and write consistent code.

---

## 2. Core Philosophy

We develop based on the following 3 principles:

1.  **Clean Architecture (Universal):**
    -   A robust structure that does not change even if the domain (e.g., Public, Finance) changes.
    -   Strict separation of `Router` -> `Service` -> `Repository/AI` layers.
2.  **Hybrid AI Strategy (Cost & Privacy):**
    -   Fluidly switch between **API (SKT A.X)** and **Local (Ollama)** models according to the situation.
    -   Achieve both cost efficiency and data security.
3.  **Agent Safety First (Antigravity Protocol):**
    -   Internalize **Guardrails** (Safety Rules) to fundamentally block accidents (deletion, leakage) that may occur when the AI agent autonomously executes code.

---

## 3. KODEX Documentation Map

This project manages guidelines for each area in a modular fashion, following the **Single Source of Truth (SSoT)** principle.

### 🏛️ Foundation
-   **`README.md` (This File):** Project overview and general guide.
-   **`rules.md` (Master Rule):** Hub for agent code of conduct and overall rules. **AI must read this file first before working.**
-   **`Architecture.md`**: System blueprint (FastAPI + HTMX + Hybrid LLM).
-   **`Folder_Standards.md`**: Directory structure and data pipeline standards.
-   **`Code_Style.md`**: Coding standards such as Python (Async), Type Hint, Naming Convention, etc.

### 🧠 AI & Domain Guide (AI Engine)
-   **`RAG.md`**: Retrieval-Augmented Generation (RAG) pipeline standards from data collection to answer generation.
-   **`Voice_Pipeline.md`**: Voice processing standards leading from STT (Whisper) → RAG → TTS.
-   **`Prompt.md`**: Library for personas, system prompts, and agent self-correction prompts.

### ⚙️ Ops & Quality
-   **`Testing_Strategy.md`**: Unit/Integration testing and AI logic verification strategy.
-   **`Deployment_CICD_Guideline.md`**: Docker (CPU/GPU), GitHub Actions, model deployment strategy.

### 🌐 Bilingual Support
-   All documents are managed in parallel in **Korean (Primary)** and **English (Secondary, `_eng.md`)**.
-   In case of conflict between contents, the **Korean document has the Source of Truth**. AI agents must perform tasks based on the Korean document.

---

## 4. Antigravity Agent Protocol (For AI Agents)

*This section is a direct command for the AI Agent (Cursor, Antigravity) reading this document.*

1.  **Safety Check:** Before executing terminal commands (`rm`, `drop`), you must request user approval.
2.  **Context Loading:** Read `rules.md` before starting work to understand the domain (Context) of the current project.
3.  **Hybrid Mode:** Always consider `SKT A.X` (Primary) and `Ollama` (Fallback) logic when writing code.
4.  **Self-Correction:** After writing code, propose verification test code according to `Testing_Strategy.md`.

---

## 5. Developer Workflow

New developers (or AI) contribute to the project in the following order:

1.  **Initialize:** Create the project skeleton by referring to `Folder_Standards.md`.
2.  **Implement:**
    -   Business logic follows the Clean Architecture of `Architecture.md`.
    -   AI features follow the specs of `RAG.md` and `Voice_Pipeline.md`.
3.  **Verify:** Secure stability by running `pytest` according to `Testing_Strategy.md`.
4.  **Deploy:** Build and deploy Docker images according to `Deployment_CICD_Guideline.md`.

---

## 6. License

`KevinCY-Antigravity` aims for the sharing and advancement of knowledge.

-   **Documentation (CC BY 4.0):** Free modification and distribution allowed under attribution (KevinCY-Kim).
-   **Code Samples (MIT License):** Unlimited code use allowed, including commercial use.

---
**Build robust, Run safe.**
*Copyright (c) 2025 KevinCY-Kim. All rights reserved.*