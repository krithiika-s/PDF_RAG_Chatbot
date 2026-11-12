# 📄 PDF Document Q&A Bot using Retrieval-Augmented Generation (RAG)

Upload PDF documents, ask questions in natural language, and get answers that are
**grounded in your documents**, together with the **document name and page number**
of every source.

> 🔗 **Live demo:https://pdfragchatbot-urhadesdvtcp2afadbya8j.streamlit.app/ 
> 🖼️ **Screenshot:** 

## Why RAG?
Fine-tuning stores knowledge in model weights: it is expensive and hard to update.
RAG (Lewis et al., 2020) keeps knowledge in an external index and retrieves it at question
time, so new documents are usable immediately, and constraining the LLM to the retrieved
text reduces hallucination and enables source attribution.

## Features
- PDF ingestion: parsing, cleaning (ligatures, hyphenation, headers/footers), page-aware chunking
- Dense embeddings (`all-MiniLM-L6-v2`) stored in a persistent **ChromaDB** index (cosine / HNSW)
- Semantic retrieval with top-k, similarity threshold and per-document filtering
- Grounded answers from any OpenAI-compatible LLM (Groq, OpenAI, Gemini, Ollama, ...)
- Source panel with document, page, similarity score, snippet, and which passages were cited
- Duplicate-upload detection, multi-PDF support, delete documents from the index
- Works without an API key (shows the best matching passage) for offline demos
- Unit tests + CI, batch evaluation script, Dockerfile

## Architecture
```
PDF ─► parse & clean ─► chunk (per page) ─► embed ─► ChromaDB
                                                        │
question ─► embed ─► top-k search ─► grounded prompt ─► LLM ─► answer + [n] citations + sources
```
Details: [`docs/architecture.md`](docs/architecture.md) · [`docs/ingestion_pipeline.md`](docs/ingestion_pipeline.md)

## Quick start
```bash
git clone <your-repo-url> && cd <repo>
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env                                    # then put your LLM_API_KEY in .env
streamlit run app.py
```
Open http://localhost:8501, upload PDFs in the sidebar, click **Index uploaded PDFs**, and ask away.
A sample document lives in `data/sample_docs/rag_intro.pdf`.

### Command line
```bash
python scripts/ingest.py data/sample_docs          # build the index
python scripts/ask.py "How does RAG reduce hallucination?"
python scripts/evaluate.py                         # writes results/eval_results.csv
```

### Getting a free LLM key
Create a key at [console.groq.com](https://console.groq.com/keys) (or use OpenAI / Gemini / a local
Ollama model) and set `LLM_API_KEY`, `LLM_BASE_URL`, `LLM_MODEL` in `.env`. See `.env.example`.

## Configuration
| Variable | Default | Meaning |
|---|---|---|
| `LLM_API_KEY` / `LLM_BASE_URL` / `LLM_MODEL` | Groq endpoint | Any OpenAI-compatible chat API |
| `EMBEDDING_BACKEND` | `sentence-transformers` | `onnx` = same model without PyTorch (light) |
| `CHUNK_SIZE` / `CHUNK_OVERLAP` | `1000` / `200` | Characters per chunk / overlap |
| `CHROMA_DIR` | `./storage/chroma` | Where the index is saved |

## Project structure
```
app.py                   Streamlit UI
src/rag/                 pdf_loader · text_cleaner · chunker · embedder · vector_store · ingestion
                         retriever · prompts · llm · pipeline · factory
scripts/                 ingest.py · ask.py · evaluate.py
tests/                   unit tests (offline: fake embedder, fake LLM)
data/sample_docs/        sample PDF            data/eval/  evaluation questions
docs/                    architecture, ingestion, deployment, team contributions
```

## Tests
```bash
python -m pytest
```

## Deployment
Streamlit Community Cloud, Hugging Face Spaces or Docker: see [`docs/deployment.md`](docs/deployment.md).

## Limitations & future work
Scanned PDFs need OCR · follow-up questions are not rewritten using chat history ·
tables/figures are plain text · ideas: cross-encoder re-ranking, hybrid BM25 + vector search, streaming.

## Results
_Run `python scripts/evaluate.py` and paste the retrieval hit-rate, keyword recall and latency here._

## Team
See [`docs/team_contributions.md`](docs/team_contributions.md).

## References
Lewis et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* NeurIPS.  
Reimers & Gurevych (2019). *Sentence-BERT.* EMNLP.

## License
MIT
