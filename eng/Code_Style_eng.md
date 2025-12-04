# KevinCY-Kodex — Code Style Guide

**Version:** 2025.11
**Scope:** Python · FastAPI · LangGraph · RAG · HTMX · Tailwind

---

## 1. General Coding Principles

-   Apply **`type hint` 100%** to all code
-   Functions must adhere to **Single Responsibility Principle (SRP)**
-   **Async First:** All DB, API, File I/O operations must use `async def` / `await` pattern
-   Name variables, functions, and files based on meaning
-   No magic numbers/strings (hard-coded values) → Separate into `settings`/`config`/constants
-   Business logic flow: `routers` → `services` → `repositories`/`ai`
-   Minimize side effects (Aim for pure functions if possible)

---

## 2. Python Code Style

### 2.1 Naming Rules

| Target | Rule | Example |
| :--- | :--- | :--- |
| Variable | `snake_case` | `user_id`, `file_path` |
| Function | `snake_case` | `load_document()`, `run_rag()` |
| Class | `PascalCase` | `DocumentParser`, `ChatService` |
| Constant | `UPPER_SNAKE_CASE` | `DEFAULT_CHUNK_SIZE = 800` |
| Filename | `snake_case` | `chat_service.py`, `rag_core.py` |

### 2.2 Formatting

-   Based on **PEP8**, 4-space indentation
-   Max line length: **100~120 characters** (Wrap if exceeded)
-   **`import` Order**:
    1.  Standard Library
    2.  Third-party Library
    3.  Local Module
    ```python
    # 1. Standard Library
    import json
    import os
    from pathlib import Path
    
    # 2. Third-party Library
    import numpy as np
    from fastapi import APIRouter
    
    # 3. Local Module
    from app.core.config import settings
    from app.services.chat_service import handle_chat
    ```
-   Do not use `from x import *`
-   Remove unused `import`s and variables
-   Minimize `try/except` and specify concrete exception types

### 2.3 Docstring Style

-   Public functions/methods use a simple summary and `Args`/`Returns` format.
    ```python
    from pathlib import Path

    def load_document(path: Path) -> str:
        """
        Loads text from the given file path.
    
        Args:
            path: Text file path.
    
        Returns:
            Full text string read from the file.
        """
    ```

---

## 3. FastAPI Code Style

### 3.1 Router File Rules

-   **Filename**: `xxx_router.py` (e.g., `chat_router.py`, `voice_router.py`)
-   **Router Instance Name**: Always `router`
    ```python
    from fastapi import APIRouter
    
    router = APIRouter(prefix="/chat", tags=["chat"])
    ```

### 3.2 Router Internal Principles

-   Router is **only responsible for I/O** and does not include business logic.
-   Follows the flow: `RequestModel` → Call `Service` → Return `ResponseModel`.
    ```python
    from app.models.chat_model import ChatRequest, ChatResponse
    from app.services.chat_service import handle_chat
    
    @router.post("/", response_model=ChatResponse)
    async def chat(request: ChatRequest) -> ChatResponse:
        return await handle_chat(request)
    ```

### 3.3 Response Rules

-   Use Pydantic models for return types whenever possible.
-   Minimize direct `dict` returns and define models if necessary.

---

## 4. Pydantic Model Style

### 4.1 Basic Rules

-   **Filename**: `xxx_model.py`
-   **Class Name**: `PascalCase`
-   Mainly used for I/O DTOs, does not include business logic.
    ```python
    from pydantic import BaseModel
    
    class ChatRequest(BaseModel):
        user_id: str | None = None
        query: str
    
    class ChatResponse(BaseModel):
        answer: str
        sources: list[str] | None = None
    ```

---

## 5. Services Layer Style

### 5.1 Rules

-   **Filename**: `xxx_service.py`
-   **Role**: Combination of business logic and workflow
-   **Main Responsibilities**:
    -   Input validation (if needed)
    -   Combination of multiple `repository`/`ai` calls
    -   Exception handling and logging
-   Maintain a structure where Router calls Service functions directly.
    ```python
    from app.models.chat_model import ChatRequest, ChatResponse
    from app.ai.graph import run_chat_graph
    
    async def handle_chat(request: ChatRequest) -> ChatResponse:
        """Entry point for service layer handling chat requests."""
        state = {"query": request.query, "user_id": request.user_id}
        result = await run_chat_graph(state)
        return ChatResponse(answer=result["answer"], sources=result.get("sources"))
    ```

---

## 6. Repositories Style

### 6.1 Rules

-   **Filename**: `xxx_repository.py`
-   **Responsibility**: **Dedicated to IO** such as DB, File, Vector DB, External API
-   Write as pure functions whenever possible, do not include business rules.
-   Perform only roles like save/retrieve/search.
    ```python
    from typing import Sequence
    from app.models.chunk_model import Chunk

    async def load_chunks(document_id: str) -> Sequence[Chunk]:
        """Asynchronously loads the list of chunks corresponding to the document ID."""
        # ... DB or File System Logic ...
        pass
    ```

---

## 7. AI / LangGraph Code Style

### 7.1 Folder Structure

AI and LangGraph related folder structure follows the project-wide standard.
> Refer to `Folder_Standards.md` for detailed AI-related folder structure.

### 7.2 LangGraph Node Function Style

-   All nodes maintain the `state: dict -> dict` form.
    ```python
    async def retrieval_node(state: dict) -> dict:
        query: str = state["query"]
        docs = retrieve_documents(query)
        return {**state, "retrieved_docs": docs}
    ```

### 7.3 LLM Call Style (Comply with Hybrid Config)

-   Utilize environment variables like `AX_MODEL_NAME` defined in Master Rule.
    ```python
    from app.core.config import settings

    async def generate_answer(context: str, question: str) -> str:
        prompt = f"""
        [Rule] Answer based on evidence, say you don't know if you don't know.
        [Context] {context}
        [Question] {question}
        """
        # Call SKT A.X or Ollama (Depending on settings)
        return await llm_client.chat(
            model=settings.AX_MODEL_NAME,
            messages=[{"role": "user", "content": prompt}],
            temperature=settings.AX_TEMPERATURE
        )
    ```

---

## 8. RAG Code Style

### 8.1 Chunking

-   **Default Settings**: `chunk_size = 800`, `chunk_overlap = 200`
    ```python
    def chunk_text(content: str, size: int = 800, overlap: int = 200) -> list[str]:
        chunks: list[str] = []
        start = 0
        length = len(content)

        while start < length:
            end = min(start + size, length)
            chunk = content[start:end]
            chunks.append(chunk)
            start += size - overlap

        return chunks
    ```

### 8.2 Retrieval

-   **Hybrid**: Combination of Keyword (`BM25`) + Vector Search
-   Separate `Rerank` into a separate function/module.
    ```python
    async def hybrid_search(query: str, k: int = 10) -> list[dict]:
        keyword_docs = await bm25_search(query, k=k)
        vector_docs = await dense_search(query, k=k)
        merged = merge_results(keyword_docs, vector_docs)
        return await rerank(query, merged)
    ```

### 8.3 Answer Generation

-   Use structured output (JSON format `dict`) whenever possible.
    ```python
    def build_answer_payload(answer: str, sources: list[str]) -> dict:
        return {
            "answer": answer,
            "sources": sources,
        }
    ```

---

## 9. Error Handling Style

### 9.1 Rules

-   FastAPI global exception handler is managed in `/app/core/exception_handlers.py`.
-   Convert exceptions to appropriate `HTTPException` in the `Service` layer.
    ```python
    from fastapi import HTTPException, status
    
    def ensure_document_exists(doc_id: str) -> None:
        if not document_exists(doc_id):
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="The document could not be found.",
            )
    ```
-   Do not record sensitive information (SSN, API Token, etc.) in logs.

---

## 10. Logging Style

-   Use structured logs (JSON format).
    ```python
    import logging
    
    logger = logging.getLogger(__name__)
    
    def log_chat_request(user_id: str | None, query: str) -> None:
        logger.info({
            "event": "chat_request",
            "user_id": user_id,
            "query": query[:100],  # Consider sensitive info and length limit
        })
    ```

---

## 11. HTMX Code Style

### 11.1 Basic Principles

-   HTMX request paths always use the `/api/*` namespace.
-   Explicitly use `hx-target`, `hx-swap`, `hx-indicator`.
    ```html
    <form
      hx-post="/api/chat"
      hx-target="#chat-window"
      hx-swap="beforeend"
      hx-indicator="#chat-loading"
    >
      <input type="text" name="query" />
      <button type="submit">Send</button>
    </form>
    ```

---

## 12. Tailwind Code Style

-   **Class Order** maintains the following group order:
    1.  Layout (`flex`, `grid`, etc.)
    2.  Alignment/Spacing (`items-center`, `gap`, `p`/`m`)
    3.  Typography (`text-sm`, `font-medium`)
    4.  Color/Background (`bg-white`)
    5.  Others (`rounded-lg`, `shadow`, `transition`)
    ```html
    <div class="flex items-center gap-2 px-4 py-2 text-sm font-medium bg-white rounded-lg shadow">
      ...
    </div>
    ```
-   Prioritize using Tailwind default palette over meaningless custom colors.

---

## 13. File Naming Rules

| File Type | Naming Example |
| :--- | :--- |
| Router | `chat_router.py` |
| Service | `chat_service.py` |
| Repository | `chat_repository.py` |
| Model | `chat_model.py` |
| Util | `string_utils.py` |
| AI Module | `retriever.py` |
| Test Code | `test_chat_service.py` |

---

## 14. Test Code Style (pytest)

### 14.1 Rules

-   **Filename**: `test_xxx.py`
-   **Test Function Name**: Use `test_` prefix
-   **Given–When–Then** pattern is recommended.
    ```python
    import pytest
    from app.models.chat_model import ChatRequest
    from app.services.chat_service import handle_chat
    
    @pytest.mark.asyncio
    async def test_handle_chat_returns_answer():
        # Given
        request = ChatRequest(query="Hello")
    
        # When
        response = await handle_chat(request)
    
        # Then
        assert response.answer
        assert isinstance(response.answer, str)
    ```

---

## 15. Git Commit Message Style

-   Use English-based `prefix`.
    -   `feat`: Add new feature (e.g., `feat: add voice chat STT pipeline`)
    -   `fix`: Fix bug (e.g., `fix: handle empty query in chat service`)
    -   `refactor`: Code refactoring (e.g., `refactor: simplify rag retrieval flow`)
    -   `docs`: Update documentation (e.g., `docs: update architecture and code-style`)
    -   `chore`: Build, dependency, etc. changes (e.g., `chore: bump dependency versions`)

---

End of KevinCY-Kodex Code Style Guide