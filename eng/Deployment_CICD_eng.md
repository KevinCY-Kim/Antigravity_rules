# KevinCY-Kodex — Deployment & CI/CD Guideline

**Version:** 2025.11.27 (Universal Standard)
**Scope:** FastAPI · Docker (CPU/GPU) · GitHub Actions · Model Ops

---

## 1. CI/CD Pipeline Overview

The goal of the `KevinCY-Kodex` CI/CD pipeline is to deliver code changes to users in a stable and automated manner.

**Overall Flow**:
`Git Push` (feature branch) → `Pull Request` → **[CI]** Run `Lint & Test` → `Code Review` & `Merge` (main branch) → **[CD]** `Docker Build & Push` → `Deploy` (staging/production)

---

## 2. Branching Strategy

We follow a simplified version of **Git-Flow**.

-   `main`: Main branch that is always in a deployable state.
-   `develop`: Branch where features being developed for the next release are integrated (Optional).
-   `feature/<feature-name>`: Branch for developing new features. Branches off from `main` or `develop`.
-   `fix/<bug-name>`: Branch for fixing bugs.

All work starts in `feature` or `fix` branches and is merged into the `main` branch via `Pull Request`.

---

## 3. CI (Continuous Integration)

We build the CI pipeline using **GitHub Actions**.

-   **Trigger**: When `push` or `pull_request` events occur on the `main` branch.
-   **Key Jobs**:
    1.  **System Dependencies**: First install system packages required for voice/AI processing (e.g., `ffmpeg`, `build-essential`).
    2.  **Linting**: Check code style using `ruff` or `flake8`.
    3.  **Testing**: Run `pytest` to ensure all tests pass (Comply with `Testing_Strategy.md`).
-   **Goal**: Guarantee that all code merged into the `main` branch has passed minimum quality standards.

**Example Workflow (`.github/workflows/ci.yml`)**:
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v3
      with:
        python-version: '3.11'
    - name: Install System Deps (Audio/AI)
      run: |
        sudo apt-get update
        sudo apt-get install -y ffmpeg
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Lint with ruff
      run: |
        pip install ruff
        ruff check .
    - name: Test with pytest
      run: |
        pip install pytest pytest-asyncio pytest-mock httpx
        pytest
```

---

## 4. Dockerization Strategy (AI Optimized)

-   **`Dockerfile` Standards**:
    -   **Base Image Strategy**:
        -   **CPU Mode (API/Web):** `python:3.11-slim` (Prioritize lightweight)
        -   **GPU Mode (Local LLM/Whisper):** `nvidia/cuda:12.x.x-base-ubuntu22.04` (CUDA support mandatory)
    -   **System Dependencies**: Must include `ffmpeg` package installation for `Voice_Pipeline` processing.
    -   **Multi-stage Build**: Minimize final image size by separating build tools (`gcc`, etc.) and execution environment.
    -   **Non-root User**: Create an `appuser` account to run the application for security.
-   **Model Weights Handling (Important)**:
    -   **Never include LLM/Embedding model files reaching several GBs in the Docker image.**
    -   Connect the host's model folder via **Volume Mount** at runtime, or download from S3/Cloud Storage in the container startup script.
-   **`.dockerignore`**: Must exclude unnecessary or temporary files like `.git`, `__pycache__`, `.env`, `data/`, `static/tts/`.

---

## 5. CD (Continuous Deployment)

We build the CD pipeline using **GitHub Actions**.

-   **Staging Environment**:
    -   **Trigger**: Automatically deployed whenever there is a `push` to the `main` branch.
    -   **Process**: Docker Image Build & Push (Registry) → `docker pull` & Service Restart on Staging Server.
-   **Production Environment**:
    -   **Trigger**: Deployed upon **Manual Execution (workflow_dispatch)** or **Git Tag** creation. (e.g., `v1.0.0`)
    -   **Process**: Deploy a verified specific version of Docker image, recommending **Rolling Update** strategy to minimize service downtime.

---

## 6. Secret Management

Never include sensitive information like API keys or DB connection info in the code.

-   **Local Environment**: Use `.env` file, and add this file to `.gitignore` to exclude it from version control.
-   **CI/CD Environment**: Inject as environment variables within the workflow (`yaml`) using **GitHub Secrets**.
-   **Deployment Environment (Staging/Production)**:
    -   Set as system environment variables on the server, or load a `.env` file from a secured path at deployment time.
    -   Recommend using **Secret Manager** services from cloud providers if possible.

---

## 7. Monitoring & Rollback

-   **Monitoring**: Continuously monitor application status (Error Rate, Latency, CPU/GPU Usage) after deployment. (Refer to logging rules in `Architecture.md` and `Voice_Pipeline.md`)
-   **Rollback**: Prepare a **Revert Script** in advance to immediately revert to the previous Docker tag version (e.g., `v1.0.0` -> `v0.9.9`) in case of critical issues after deployment.

---

## 8. Agent Protocol (Antigravity Deployment Guardrails)
*Absolute rules for agents to follow when writing deployment-related scripts (Dockerfile, YAML).*

1.  **No Hardcoding:** Do not hardcode API Keys or Passwords as text inside scripts. Must use `${ENV_VAR}` format.
2.  **Volume Check:** When configuring deployment, ensure `data/` (Vector DB) and `static/` (TTS files) folders are connected to **Persistent Storage (Volume)**. (Prevent data loss upon container restart)
3.  **Dry Run:** Before executing deployment commands, show the target server and version to the user and request approval.

End of KevinCY-Kodex Deployment & CI/CD Guideline