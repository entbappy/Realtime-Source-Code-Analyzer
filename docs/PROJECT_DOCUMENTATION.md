# Project Documentation

## Overview

Realtime Source Code Analyzer is a developer tool that combines Git repository ingestion, OpenAI embeddings, and LangChain retrieval to let users ask questions about source code in realtime.

The application is built with:

- Flask for the web UI and API endpoints
- LangChain for document loading, text splitting, embeddings, and conversational retrieval
- OpenAI Embeddings and ChatOpenAI for semantic search and natural language answers
- Chroma for local vector storage and persistence

## Features

- Clone a Git repository from a URL
- Load Python source files from the local clone
- Split library code into chunks for accurate semantic retrieval
- Generate embeddings and persist them in a local Chroma database
- Query the repository content with a natural language chat interface

## Architecture

Key components:

- `app.py` — main API and web server
- `src/helper.py` — ingestion, parsing, chunking, embeddings
- `store_index.py` — creates and persists the vector database
- `templates/index.html` — frontend UI
- `db/` — persisted Chroma database files
- `Assets/` — architecture screenshots
- `workflows/` — Mermaid diagrams for system workflows

## Setup

1. Clone the repository.
2. Create a Python virtual environment.
3. Install dependencies from `requirements.txt`.
4. Create a `.env` file containing `OPENAI_API_KEY`.
5. Run `python app.py`.

### Example

```bash
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
python app.py
```

## Optional UV Package Manager Support

For a faster installation and run workflow, use the `uv` package manager. It simplifies dependency installation and app execution with a lightweight wrapper.

```bash
pip install uv
uv install
uv run python app.py
```

## Workflows

- `workflows/app_flow.md` — how HTTP requests are handled
- `workflows/ingestion_flow.md` — repository ingestion and indexing process
- `workflows/architecture_flow.md` — overall architecture diagram

## How it works

1. The UI sends questions to Flask endpoints.
2. The `/chatbot` endpoint clones a repository and rebuilds the index.
3. `store_index.py` loads repo files, splits them, creates embeddings, and saves to Chroma.
4. The `/get` endpoint uses a conversational retrieval chain to answer questions.

## Future improvements

- Support more languages beyond Python
- Add authentication for API endpoints
- Add a more polished frontend UI
- Add support for multiple repositories and incremental updates
- Add tests and CI workflows
