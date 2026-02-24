# Multi-Agent Daily Digest

A multi-agent AI system that turns raw text files into an organized daily digest. Four containerized Python agents run sequentially via Docker Compose:

1. **Ingestor** — Reads and combines raw files from `data/input/` into `ingested.txt`
2. **Summarizer** — Calls an LLM (OpenAI `gpt-4o-mini`) to distill key bullet points
3. **Prioritizer** — Scores items by urgency keywords and sorts by priority
4. **Formatter** — Produces the final Markdown report at `output/daily_digest.md`

## Prerequisites

- Python 3.10+
- Docker Desktop (Engine 20.10+)
- Docker Compose v2
- An OpenAI API key

## Setup

1. Clone this repo
2. Copy your API key into `.env`:
   ```
   OPENAI_API_KEY=sk-your-key-here
   ```
3. Add your raw text files to `data/input/`

## Run

```bash
docker compose up --build
```

The final digest will appear at `output/daily_digest.md`.

## Test

```bash
pip install pytest
python -m pytest tests/ -v
```

## Architecture

Agents communicate via a shared Docker volume (`/data`). Docker Compose enforces execution order using `depends_on: condition: service_completed_successfully`. Only the Summarizer calls an external API — all other agents are pure deterministic Python.

Based on: [How to Build and Deploy a Multi-Agent AI System with Python and Docker](https://www.freecodecamp.org/news/build-and-deploy-multi-agent-ai-with-python-and-docker/) by Balajee Asish Brahmandam.
