# Realtime Source Code Analyzer

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Documentation](https://img.shields.io/badge/docs-available-brightgreen.svg)](docs/PROJECT_DOCUMENTATION.md)
[![Status](https://img.shields.io/badge/status-alpha-orange.svg)](README.md)

A lightweight Flask-based realtime source code analyzer that ingests a Git repository, builds an OpenAI-backed embedding index with LangChain and Chroma, and answers developer questions from code.

## 🚀 Key Features

- Clone and ingest Python-based Git repositories from a URL
- Split repository source files into contextual chunks
- Build and persist a Chroma vector database for fast semantic retrieval
- Query code with a conversational LLM chain via OpenAI
- Minimal Flask UI for interactive question/answer sessions
- Optional architecture diagrams and workflows in `workflows/`

## 📁 Project Structure

- `app.py` — Flask application, routes, and conversational query handling
- `src/helper.py` — repository ingestion, document loading, chunking, and embeddings
- `store_index.py` — index builder that persists a Chroma database from repo content
- `templates/` — HTML UI for the chat frontend
- `static/` — front-end static assets
- `Assets/` — architecture screenshots and visuals
- `docs/` — project documentation
- `workflows/` — Mermaid workflow diagrams for components

## 🧩 Architecture Overview

![Project Architecture](Assets/Generated_image342423%20(1).png)

> Use `docs/PROJECT_DOCUMENTATION.md` for full technical documentation and `workflows/` files for component workflows.

## 🛠️ Installation

### Option 1: Standard virtual environment

```bash
python -m venv .venv
source .venv/Scripts/activate  # Windows
pip install -r requirements.txt
```

### Option 2: Conda environment (optional)

```bash
conda create -n llmapp python=3.10 -y
conda activate llmapp
pip install -r requirements.txt
```

### Optional: Faster setup with the `uv` package manager

For a faster installation workflow, you can use the `uv` package manager to install dependencies and run the app. This is especially useful when you want a quicker setup experience.

```bash
pip install uv
uv install
uv run python app.py
```

## ⚙️ Configuration

Create a `.env` file in the project root with the following content:

```env
OPENAI_API_KEY=your_openai_api_key_here
```

## ▶️ Run the application

```bash
python app.py
```

Then open `http://localhost:8080` in your browser.

## 📘 Documentation

- Detailed project docs: `docs/PROJECT_DOCUMENTATION.md`
- Component workflows: `workflows/app_flow.md`, `workflows/ingestion_flow.md`, `workflows/architecture_flow.md`

## 💡 Notes

- This project uses OpenAI embeddings and `langchain` for semantic search.
- The vector DB is persisted in `db/` via Chroma.
- The current UI is a simple Flask page; backend routes handle ingestion and chat retrieval.

## 📦 License

This project is licensed under the MIT License. See `LICENSE` for details.
