# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-agent AI system that converts raw text files into a prioritized daily digest. Four containerized Python agents run sequentially via Docker Compose, communicating through a shared Docker volume (`/data`).

Pipeline: **Ingestor → Summarizer → Prioritizer → Formatter**

## Commands

### Run the full pipeline
```bash
docker compose up --build
```
Output appears at `output/daily_digest.md`.

### Run tests
```bash
python -m pytest tests/ -v
```

### Run a single test
```bash
python -m pytest tests/test_prioritizer.py::test_urgent_keyword_scores_one -v
```

## Architecture

### Agent Pipeline (sequential, Docker Compose `depends_on`)

| Agent | Location | Purpose | Dependencies |
|-------|----------|---------|-------------|
| Ingestor | `agents/ingestor/` | Reads all files from `data/input/`, concatenates into `data/ingested.txt` | stdlib only |
| Summarizer | `agents/summarizer/` | Calls OpenAI `gpt-4o-mini` to produce bullet-point summary → `data/summary.txt` | `openai` package |
| Prioritizer | `agents/prioritizer/` | Scores lines by urgency keywords, sorts descending → `data/prioritized.txt` | stdlib only |
| Formatter | `agents/formatter/` | Generates final Markdown digest → `output/daily_digest.md` | stdlib only |

### Data Flow

Agents communicate exclusively via files on a shared Docker volume. No message queues or APIs between agents. Each agent reads its predecessor's output file and writes its own.

`data/input/*.txt` → `data/ingested.txt` → `data/summary.txt` → `data/prioritized.txt` → `output/daily_digest.md`

### Key Design Details

- Only the Summarizer calls an external API; all other agents are pure deterministic Python
- Summarizer has retry logic (3 retries, exponential backoff) and a 512M memory limit
- Prioritizer keywords: "urgent", "today", "asap", "important", "deadline", "critical", "action required" (case-insensitive)
- Each agent has its own Dockerfile (Python 3.10-slim base)
- Agents run once and exit (no restart policy)

## Environment

Requires a `.env` file at project root with `OPENAI_API_KEY=sk-...` (injected into summarizer container via docker-compose).

## Test Structure

Tests are in `tests/` using pytest. Currently only the prioritizer's `score_line()` function is tested. Tests import directly from `agents/prioritizer/app.py` via sys.path manipulation.
