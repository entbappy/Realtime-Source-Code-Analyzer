# Source Code Analyzer — Project Draft

## 1. Project Overview

**Source Code Analyzer** is a generative AI–powered application that lets users point a GitHub repository at the system and then ask natural-language questions about that codebase. The system clones the repo, indexes Python source files, embeds them in a vector store, and answers questions using retrieval-augmented generation (RAG) with conversational memory.

**Target users:** Developers, researchers, and technical reviewers who want to explore or understand unfamiliar Python repositories without reading every file.

---

## 2. Problem Statement & Goals

### Problem
- Understanding large or unfamiliar codebases is time-consuming.
- Static search (e.g., grep) does not capture semantics or intent.
- Documentation is often missing or outdated.

### Goals
- **Ingest** a GitHub repository (clone and load Python source files).
- **Index** code into a vector database for semantic search.
- **Answer** user questions in natural language using the indexed code as context.
- **Maintain** conversation context so follow-up questions make sense.

---

## 3. System Architecture

### High-Level Flow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  User (Browser) │────▶│  Flask Web App    │────▶│  OpenAI API     │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                          │                        │
        │                          ▼                        │
        │                 ┌──────────────────┐              │
        │                 │  ChromaDB        │              │
        │                 │  (Vector Store)   │              │
        │                 └──────────────────┘              │
        │                          ▲                        │
        │                          │                        │
        └──────────────────────────┼────────────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │  Ingestion & Indexing       │
                    │  (Git clone → Load → Split  │
                    │   → Embed → ChromaDB)       │
                    └────────────────────────────┘
```

### Components

| Component | Role |
|-----------|------|
| **Flask app** | Web server, routes, session handling; serves UI and chat/ingestion APIs. |
| **GitPython** | Clones the user-provided GitHub repository into a local `repo/` directory. |
| **LangChain** | Document loading (GenericLoader + LanguageParser), text splitting, embeddings, and RAG chain. |
| **ChromaDB** | Persistent vector store for code chunks; used for similarity search. |
| **OpenAI** | Embeddings (e.g., text-embedding-ada-002) and chat model (e.g., GPT) for generation. |

---

## 4. Technology Stack

| Layer | Technology |
|-------|------------|
| **Backend** | Python 3.10, Flask |
| **AI / ML** | LangChain, OpenAI API (embeddings + chat) |
| **Vector DB** | ChromaDB 0.4.4 |
| **Repo access** | GitPython |
| **Frontend** | HTML, Bootstrap, jQuery |
| **Config** | python-dotenv, `.env` for `OPENAI_API_KEY` |

### Dependencies (from `requirements.txt`)

- `openai==0.28`
- `tiktoken`
- `chromadb==0.4.4`
- `langchain==0.0.249`
- `flask`
- `GitPython`
- `python-dotenv`
- Local package: `-e .` (installable via `setup.py`)

---

## 5. Data Flow & Pipeline

### 5.1 Repository Ingestion & Indexing

1. **User input:** User submits a GitHub repository URL (e.g., via the “Give Github Repository Link Here” form).
2. **Clone:** `repo_ingestion(repo_url)` creates `repo/` and clones the repo with GitPython.
3. **Load:** `load_repo("repo/")` uses LangChain’s `GenericLoader` + `LanguageParser` to load **Python files only** (`.py`), with a parser threshold of 500 characters.
4. **Split:** `text_splitter(documents)` uses `RecursiveCharacterTextSplitter` (Python-aware) with chunk size 2000 and overlap 200.
5. **Embed:** OpenAI embeddings are computed for each chunk.
6. **Store:** Chunks and embeddings are written to ChromaDB with `persist_directory='./db'`.
7. **Trigger:** Ingestion can be triggered from the UI; the app then runs `store_index.py` to refresh the index (e.g., `os.system("python store_index.py")`).

### 5.2 Chat / Q&A

1. **User message:** User types a question in the chat interface.
2. **Retrieval:** `ConversationalRetrievalChain` uses the ChromaDB retriever (MMR, `k=8`) to fetch relevant code chunks.
3. **Memory:** `ConversationSummaryMemory` keeps a summarized chat history for multi-turn context.
4. **Generation:** The chain passes context + history to the OpenAI chat model and returns an answer.
5. **Special command:** Input `"clear"` triggers cleanup of the `repo/` directory (implementation may use `rm -rf repo`, which is Unix-specific).

---

## 6. Key Features

- **GitHub repo ingestion:** Clone any public GitHub repository and index its Python code.
- **Python-only parsing:** Focus on `.py` files with language-aware parsing and splitting.
- **Semantic search:** Vector similarity (via ChromaDB) instead of keyword-only search.
- **Conversational Q&A:** Multi-turn chat with summarized history.
- **Simple web UI:** One page for repo URL submission; another view for chat.
- **Persistent index:** ChromaDB persisted under `./db` so the app can reuse the index across restarts (until repo is changed and re-indexed).

---

## 7. Project File Structure

```
Source_Code_Analysis/
├── app.py                 # Flask app: routes (/ , /chatbot, /get), RAG chain, ChromaDB load
├── store_index.py         # Script: load repo → split → embed → save to ChromaDB
├── template.py            # Utility: creates default dirs/files (e.g. src/, .env, requirements.txt)
├── setup.py               # Package setup (Generative AI Project, find_packages)
├── requirements.txt       # Python dependencies
├── .env                   # OPENAI_API_KEY (not committed)
├── .gitignore
├── README.md
├── LICENSE
├── db/                    # ChromaDB persistence (created at runtime)
├── repo/                  # Cloned GitHub repo (created at runtime)
├── src/
│   ├── __init__.py
│   ├── helper.py          # repo_ingestion, load_repo, text_splitter, load_embedding
│   └── prompt.py          # Optional prompt template (context + question → answer)
├── templates/
│   └── index.html         # Combined UI: repo URL form + chat interface
├── static/
│   ├── style.css          # Chat layout and styling
│   └── jquery.min.js      # Client-side AJAX
└── research/
    └── Source_Code_Analysis.ipynb   # Research/experimentation notebook
```

---

## 8. API Endpoints

| Method | Route | Purpose |
|--------|--------|---------|
| GET / POST | `/` | Serves the main page (repo URL form + chat UI). |
| POST | `/chatbot` | Accepts `question` (GitHub repo URL); runs ingestion and `store_index.py`; returns JSON with the submitted URL. |
| POST | `/get` | Accepts `msg` (user message); runs RAG chain; returns plain-text answer. Special: `msg == "clear"` clears `repo/`. |

---

## 9. Configuration & Environment

- **Python:** 3.10 recommended (e.g., conda env `llmapp`).
- **Environment variable:** `OPENAI_API_KEY` in `.env` (root directory).
- **Port:** App runs on `http://0.0.0.0:8080` (all interfaces, debug mode).

### Setup Steps (from README)

1. Clone the project repository.
2. Create conda env: `conda create -n llmapp python=3.10 -y` and activate.
3. Install: `pip install -r requirements.txt`.
4. Add `OPENAI_API_KEY` to `.env`.
5. Run: `python app.py`.
6. Open browser to the displayed host/port (e.g., `http://localhost:8080`).

---

## 10. Known Limitations & Considerations

- **Python only:** Only `.py` files are loaded; other languages are ignored.
- **Single repo:** The design assumes one active repo at a time; re-submitting a new URL re-clones and re-indexes.
- **OS dependency:** The “clear” action uses `rm -rf repo`, which is Unix/Linux/macOS; would fail on Windows without adaptation.
- **Subprocess:** Ingestion triggers `os.system("python store_index.py")`, which blocks and depends on `python` being in PATH.
- **No auth:** No authentication or rate limiting on endpoints.
- **Cost:** All embeddings and chat calls go to OpenAI; cost scales with repo size and usage.
- **Parser threshold:** `parser_threshold=500` may skip very small files; tuning may be needed for tiny scripts.

---

## 11. Possible Future Enhancements

- **Multi-language support:** Extend `LanguageParser` and loaders for JavaScript, TypeScript, Java, etc.
- **Incremental indexing:** Only re-process changed files or new commits.
- **Repository selector:** Support multiple repos and let the user choose which one to query.
- **Cross-platform clear:** Use `shutil.rmtree("repo")` (or equivalent) instead of `rm -rf`.
- **Async ingestion:** Run indexing in a background task/queue so the UI does not block.
- **Custom prompts:** Use the template in `src/prompt.py` explicitly in the chain for consistent formatting.
- **Caching:** Cache embeddings for unchanged files to reduce API cost.
- **Auth & security:** Add API keys, rate limits, and input validation for repo URLs.
- **Testing:** Unit tests for `helper.py`, integration tests for ingestion and one-shot Q&A.

---

## 12. Research & Experimentation

- **Notebook:** `research/Source_Code_Analysis.ipynb` is used for trials and experimentation alongside the main app.
- **Template script:** `template.py` helps scaffold the project layout (e.g., empty `src/`, `.env`, `requirements.txt`) for new setups.

---

## 13. Summary

Source Code Analysis is a RAG-based code Q&A system that:

1. Takes a GitHub repo URL and clones it.
2. Loads and chunks Python source with LangChain.
3. Builds a ChromaDB vector index with OpenAI embeddings.
4. Serves a Flask UI for repo submission and chat.
5. Answers questions using a conversational retrieval chain with OpenAI and persistent ChromaDB.

The project is suitable as a research prototype or internal tool for understanding Python codebases via natural language. Production use would benefit from the enhancements and hardening outlined above.

---

*Document version: 1.0 — Generated as project draft for Source_Code_Analysis.*
