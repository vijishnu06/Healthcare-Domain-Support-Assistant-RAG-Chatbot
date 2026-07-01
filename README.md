# Healthcare Domain Support Assistant — RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot that answers patient and visitor healthcare questions strictly from a curated set of trusted documents (CDC, WHO, NHS, MedlinePlus). Built with LangChain, OpenAI, and FAISS.

## Overview

This project implements an end-to-end RAG pipeline: it ingests healthcare PDFs/text files, chunks and embeds them, stores them in a FAISS vector index, and serves answers through a conversational retrieval chain with memory. The assistant only answers from the provided context and explicitly declines to answer out-of-scope questions.

## Features

- **Document ingestion** — loads PDF and TXT files and splits them into overlapping chunks for retrieval
- **Vector search** — FAISS index built from OpenAI embeddings, top-k similarity retrieval
- **Conversational memory** — sliding-window memory (last 6 turns) for natural follow-up questions
- **Grounded answers** — strict system prompt that refuses to answer when information isn't in the retrieved context
- **Source citation** — every response lists the source document(s) used
- **Bonus** — Streamlit UI (`app.py`) included as an additional deliverable

## Tech Stack

- [LangChain](https://python.langchain.com/) (`langchain`, `langchain-community`, `langchain-openai`)
- OpenAI API (`gpt-4o-mini` for generation, OpenAI embeddings)
- [FAISS](https://github.com/facebookresearch/faiss) for vector storage/retrieval
- `pypdf` for PDF parsing
- Streamlit (bonus UI)

## Documents Used

All source documents are publicly available:

| File | Source |
|------|--------|
| `cdc_diabetes_guide.pdf` | https://www.cdc.gov/diabetes |
| `cdc_hypertension.pdf` | https://www.cdc.gov/bloodpressure |
| `who_patient_safety.pdf` | https://www.who.int/news-room/fact-sheets/detail/patient-safety |
| `nhs_discharge_guidance.pdf` | https://www.england.nhs.uk |
| `medlineplus_medications.txt` | https://medlineplus.gov |

## Notebook Structure

| Section | Description |
|---|---|
| **A** | Install dependencies |
| **B** | API credentials (Colab Secrets) |
| **C** | Upload documents & ingestion (load → split → embed → FAISS index) |
| **D** | RAG chain implementation (retriever, system prompt, conversational chain) |
| **E** | Sample test conversations |

## Getting Started

### Run in Google Colab (recommended)

1. Open `Week15_RAG_Healthcare_Colab.ipynb` in Google Colab.
2. In the left sidebar, open the 🔑 **Secrets** panel and add:
   - `OPENAI_API_KEY` — your API key
   - `OPENAI_BASE_URL` — your API base URL
   
   Toggle notebook access **ON** for both.
3. Run the cells in order (Sections A → E).
4. When prompted in Section C, upload your healthcare documents (PDF/TXT).

### Run locally

```bash
pip install langchain langchain-community langchain-openai openai faiss-cpu pypdf python-dotenv tiktoken streamlit
```

Set environment variables instead of using Colab Secrets:

```bash
export OPENAI_API_KEY="your-key-here"
export OPENAI_BASE_URL="your-base-url-here"
```

Place your source documents in a `docs/` folder, then adapt the notebook cells (skip the Colab-specific `google.colab` imports for file upload and secrets) or run the equivalent Python script.

## How It Works

1. **Ingestion** — Documents in `docs/` are loaded (`PyPDFLoader` / `TextLoader`), split into ~800-character chunks with 100-character overlap using `RecursiveCharacterTextSplitter`.
2. **Indexing** — Chunks are embedded with `OpenAIEmbeddings` and stored in a local FAISS index (`faiss_index/`).
3. **Retrieval** — At query time, the top-4 most relevant chunks are retrieved via similarity search.
4. **Generation** — A strict system prompt instructs `gpt-4o-mini` to answer **only** from retrieved context, citing sources, and to reply with a fixed refusal message when the answer isn't present.
5. **Memory** — `ConversationBufferWindowMemory` retains the last 6 turns, enabling natural follow-up questions.

## Example Interactions

- ✅ **Direct question**: "What are the common symptoms of type 2 diabetes?"
- ✅ **Follow-up (memory test)**: "Does this condition affect children as well?"
- ✅ **Cross-document question**: "What does the WHO say about patient safety in hospitals?"
- ❌ **Out-of-scope (should refuse)**: "What is the best stock to invest in right now?"
- ✅ **Multi-turn**: hypertension lifestyle changes → specific sodium limit follow-up
- ✅ **Discharge guidance**: "What should a patient do after being discharged from hospital?"

## Evaluation Summary

| Requirement | Result |
|---|---|
| Answers only from loaded documents | PASS |
| Source citation per response | PASS |
| Follow-up / conversation memory | PASS |
| Refuses out-of-scope questions | PASS |
| End-to-end RAG pipeline | PASS |
| Minimum 3–5 documents | PASS (5 documents) |

## Disclaimer

This assistant is a demonstration/educational project and is **not** a substitute for professional medical advice, diagnosis, or treatment. Always consult a qualified healthcare provider for medical concerns.

## License

This project is provided for educational purposes. Refer to the original document sources (CDC, WHO, NHS, MedlinePlus) for their respective usage terms.
