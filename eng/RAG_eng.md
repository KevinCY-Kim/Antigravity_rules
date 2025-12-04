# KevinCY-Kodex — RAG Guideline
**Version:** 2025.11.27 (Universal Standard)
**Scope:** RAG Engine · Hybrid Retrieval · Hybrid LLM (API/Local) · Document QA

---

## 1. RAG Overview

RAG (Retrieval-Augmented Generation) consists of the following 4 steps:

1.  **Ingestion**: Document collection and preprocessing (Cleaning & Metadata Tagging)
2.  **Chunking & Embedding**: Semantic unit division and vectorization
3.  **Retrieval**: Hybrid Search (BM25 + Dense + Late Rerank)
4.  **Answer Generation**: LLM-based response generation (Including Reasoning Trace)

All steps are designed as independent modules and orchestrated via `LangGraph` in the `Service` layer.

---

## 2. RAG Architecture Structure

The folder structure of RAG-related modules follows the project-wide standard. Each step must adhere to the **Single Responsibility Principle (SRP)**.
> Refer to `Folder_Standards.md` for detailed AI-related folder structure.

---

## 3. Ingestion Rules

### 3.1 Supported Document Formats
-   **Structured:** CSV, JSON, Excel
-   **Unstructured:** PDF, DOCX, HWP (Dedicated parser required), HTML
-   **Domain Spec:** Public institution/Company regulation documents, Technical manuals

### 3.2 Ingestion Basic Rules
-   **Cleaning:** Must remove unnecessary noise (Table of Contents, Page Numbers, Headers/Footers, Watermarks) after text extraction.
-   **Metadata Injection:** Mandatory tagging of the following metadata for all chunks:
    -   `source`: Filename or Source URL
    -   `doc_type`: Document Type (e.g., Regulation, Manual, Notice)
    -   `section`: Section/Table of Contents info within the document
    -   `page`: Original page number (Ensure ease of verification)
    -   `date`: Document creation date or last modified date

### 3.3 Domain Specific Preprocessing (Universal)
-   **Context Isolation:** If information from other institutions irrelevant to the current project domain (e.g., Paju City, Samsung Electronics) is included, set `exclude_flag` or comment it out separately.
-   **Entity Extraction:** Extract phone numbers, department names, and person in charge information separately in the form of `meta={"contact": "...", "dept": "..."}` and save to fields.

---

## 4. Chunking Strategy

### 4.1 Basic Spec
-   **Default Size:** `800` tokens
-   **Overlap:** `200` tokens
-   **Flexibility:** Elastically adjustable depending on document nature (Law clause vs Manual).

### 4.2 Division Principles (Semantic Chunking)
-   Prioritize **Semantic Unit Division (Paragraph, Table, Section)** over simple length-based division.
-   **Table Handling:** Convert table data to markdown format or manage as a separate chunk type (`TableChunk`).
-   **Noise Reduction:** For OCR or STT results, go through a Spell Check step before chunking.

### 4.3 Chunk Schema
-   A single `chunk` object has the following fields:
    -   `id` (UUID), `text` (Content), `embedding` (Vector), `metadata` (Dict)

---

## 5. Embedding Rules

### 5.1 Basic Models
-   **Primary:** `jhgan/ko-sbert-nli` (Korean Specialized)
-   **Secondary:** `intfloat/e5-base` (Universal/English Mixed)
-   **Multilingual:** `intfloat/multilingual-e5-base`

### 5.2 Embedding Policy
-   All `chunk`s go through a Normalization process before embedding.
-   **Filter:** Exclude chunks that are too short (e.g., less than 10 characters) or meaningless from embedding targets.

### 5.3 Storage (Vector Store)
-   **Dev/Local:** `ChromaDB` or `FAISS` (Local File)
-   **Prod:** `Milvus` or `Pgvector`
-   **Management:** ID and metadata mapping are dual-managed with separate `JSONL` or RDBMS.

---

## 6. Retrieval Rules

### 6.1 Hybrid Retrieval Strategy
1.  **Keyword Search (Sparse):** Exact term matching with `BM25` algorithm (Secure Recall).
2.  **Vector Search (Dense):** Similarity search based on embedding (Secure Semantic Context).
3.  **Ensemble:** Generate Top-N candidates by weighted summation of the above two results (Reciprocal Rank Fusion, etc.).

### 6.2 Late Reranking
-   **Process:** Perform precise Rerank with `Cross-Encoder` model on the initially retrieved Top-N (e.g., 20) documents.
-   **Output:** Finally deliver only Top-K (3~5) chunks with high quality scores (`score`) to LLM.

### 6.3 Retrieval Quality Rules
-   **Domain Filter:** Prioritize applying metadata filtering so that documents from other domains are not mixed.
-   **Deduplication:** Remove duplicate chunks with over 95% content similarity, leaving only one.

---

## 7. Answer Generation Rules (The "LLM" Phase)

### 7.1 Prompt Policy (Hybrid Strategy)
*Comply with settings in `Code_Style.md` and `rules.md`.*
-   **API Mode (Primary):** Use `SKT A.X-4.0` (Env: `AX_MODEL_NAME`). Suitable for complex reasoning and high-quality writing.
-   **Local Mode (Fallback):** Use `Ollama` (Llama 3, Mistral). Suitable for simple summary or secure data processing.

### 7.2 Response Generation Principles & Constraints
**Role:** Assign "Document-based Answer Expert" role in System Prompt.
-   **Constraints:**
    -   *"Answer only within the given context."* (Prevent Hallucination)
    -   *"Must cite the sentence that is the basis of the answer."* (Citation)
    -   *"Say you don't know if you don't know."* (Honesty)

### 7.3 Output Schema (JSON)
```json
  {
    "answer": "Generated final answer text",
    "sources": ["doc_id_1#chunk_2", "doc_id_3#chunk_5"],
    "reasoning": "Logical basis for deriving the answer (Internal use)",
    "followup": ["Related additional question 1", "Related additional question 2"]
  }
```

---

## 8. Post-Processing Rules

### 8.1 Text Cleaning
-   Text Cleaning: Refine duplicate sentences, unnecessary conjunctions, and mechanical tone naturally.
-   Safety Filter: Final check for inclusion of profanity or personal information (PII).

---

## 9. LangGraph Workflow Rules

### 9.1 RAG Pipeline Graph Configuration
-   **Mandatory Nodes**: `preprocess_node`, `retrieval_node`, `rerank_node`, `generation_node`, `postprocess_node`
-   **Flow Control**: Connect general conversations unnecessary for RAG directly to LLM via `router_node`.

### 9.2 State Structure
-   **Core State**:
    -   `query`: User question (str)
    -   `user_meta`: User info (dict, optional)
    -   `retrieved_docs`: Initial search results (list[Document])
    -   `reranked_docs`: Final documents after Rerank (list[Document])
    -   `answer`: Generated answer (str)
    -   `sources`: List of source IDs (list[str])

### 9.3 Error Handling
-   **Retrieval Failure**: Return `empty list ([])` instead of error upon search failure so logic doesn't break.
-   **No Context**: If there is no valid `chunk`, skip LLM call and immediately return "No basis in document" message.
-   **System Error**: Generate pre-defined `fallback` response upon pipeline internal error.

---

## 10. Evaluation Rules

### 10.1 Quality Metrics
-   `Recall@K`, `Precision@K`: Search accuracy measurement
-   `Rerank Score`: Sorting performance of Reranking model
-   `Answer Faithfulness`: Measure if answer is based on document content (Hallucination detection)
-   `Hallucination Score`: Whether factually incorrect content is included

### 10.2 Automated Test (CI/CD)
-   **Golden Set**: Periodically compare with `Known-answer` (Question set with correct answers) baseline.
-   **Coverage**: Check if `Critical keyword` is included in search results.
-   **Alignment**: Verify consistency between retrieved `Chunk` and generated `Answer`.

---

## 11. Production Rules

### 11.1 Index Versioning & Safety
-   Manage by applying `index_version=YYYYMMDD_vX` tag for every `ingest` task.
-   **Deletion Guard**: Delete old indexes, but agents must receive user approval before index deletion (drop) tasks.
-   In principle, perform Full Re-indexing when documents change.

### 11.2 Monitoring
-   **Retrieval hit ratio**: Search success rate
-   **LLM latency**: Answer generation time
-   **Pipeline latency**: Total pipeline (including STT/TTS) latency
-   **Traffic**: Analysis of question volume by user section/time zone

### 11.3 Data Security
-   **Masking**: Sensitive information (PII) such as phone numbers, resident registration numbers, passport numbers must be **Masked** before embedding and saving.
    -   *Example:* `010-1234-5678` -> `010-****-****`

End of KevinCY-Kodex RAG Guideline