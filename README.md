# Hybrid-RAG

A local-first hybrid Retrieval-Augmented Generation pipeline that combines vector search with web fallback. Runs entirely on your machine using Ollama and Weaviate — no API keys or cloud services required.

## How It Works

Hybrid-RAG ingests documents into a Weaviate vector store, then uses an LLM-based query router to decide how to answer each question:

- **LOCAL** — retrieves relevant chunks from the vector store via similarity search
- **WEB** — falls back to DuckDuckGo search + Playwright scraping when the query falls outside ingested documents

Retrieved context is fed to a local LLM (llama3 via Ollama) to synthesize a final answer.

## Architecture

```
User Query
    │
    ▼
┌──────────────┐
│ Query Router  │  LLM classifies: LOCAL or WEB
│  (llama3)     │
└──────┬───────┘
       │
  ┌────┴────┐
  ▼         ▼
LOCAL      WEB
  │         │
  ▼         ▼
Weaviate   DuckDuckGo search
similarity  + Playwright scrape
search      + LLM synthesis
  │         │
  └────┬────┘
       ▼
  LLM Answer
   (llama3)
```

## Tech Stack

| Component | Technology |
|-----------|------------|
| LLM | Ollama — llama3 (local) |
| Embeddings | nomic-embed-text via Ollama |
| Vector Store | Weaviate v1.27 (Docker) |
| Framework | LangChain (langchain_ollama, langchain_weaviate, langchain_core, langchain_community) |
| Chunking | RecursiveCharacterTextSplitter (1000 chars, 100 overlap) |
| Web Search | DuckDuckGo + Playwright headless browser |
| Interface | CLI chat loop |

## Supported Document Formats

- **PDF** — via PyPDFLoader
- **XML** — via BeautifulSoup
- **Web pages** — via Playwright headless scraping

Documents are split into chunks with source metadata preserved for filtered retrieval.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) (for Weaviate)
- [Ollama](https://ollama.ai/) with models pulled:
  ```bash
  ollama pull llama3
  ollama pull nomic-embed-text
  ```
- Python 3.10+

## Getting Started

1. **Start Weaviate:**
   ```bash
   docker-compose up -d
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Ingest your documents** into the vector store (place PDFs, XML files, or URLs in the ingestion pipeline).

4. **Run the chat interface:**
   ```bash
   python main.py
   ```

## Project Scope

This is a ~600 LOC pipeline across 12 Python files. It demonstrates a functional hybrid RAG pattern — local vector retrieval with intelligent web fallback — suitable as a foundation for document Q&A projects. The ingestion pipeline is generic and works with any PDF, XML, or web content.

## Limitations

- CLI-only interface (no web UI)
- No automated test suite
- Weaviate runs via Docker Compose; no managed/cloud deployment config
- Single-user, single-session design

## License

See [LICENSE](LICENSE) for details.
