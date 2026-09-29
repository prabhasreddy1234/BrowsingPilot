# Jev Decision Benchmark

A local benchmark and browser automation platform for comparing TypeSafe Jev decision-making against a standard LLM for tool selection and web-grounded task execution.

The core idea is straightforward:

> Jev makes the decision. The LLM writes the answer. Your code owns the control flow.

This architecture makes it easier to evaluate whether a typed, probabilistic decision layer improves tool selection quality, latency, and operating cost while preserving the flexibility of an LLM for reasoning and synthesis.

## Overview

This repository includes:

- a Python FastAPI backend for benchmark execution, SQLite persistence, and event-driven agent workflows
- a React + TypeScript frontend for live monitoring and dashboard analysis
- Playwright-based browser automation for grounded search and browsing tasks
- side-by-side comparison flows for Jev + LLM versus LLM-only execution paths

The project is intended for local experimentation, demos, and model benchmarking rather than production deployment.

## What Jev is

Jev is TypeSafe's System One decision model. Instead of producing a free-form text completion, it returns a structured, probabilistic decision with:

- a selected tool
- confidence scores
- per-option probabilities
- low-latency execution
- input-token pricing with output cost excluded

By default, the project assumes a Jev rate of $0.042 per 1M input tokens and can run in simulation mode when no TypeSafe API key is configured.

## Key capabilities

- benchmark dashboard comparing Jev and LLM performance on labeled tool-selection tasks
- live reporting for latency, cost, and accuracy
- historical run tracking in SQLite
- browser-driven task execution for search and page retrieval workflows
- visible browser mode for demos and headless mode for comparison runs
- streaming execution events with cumulative timing, token usage, and spend
- graceful fallbacks when browser automation is unavailable
- typo-tolerant domain handling for common cases such as Wikipedia and Bing

## System architecture

The application separates decision-making from answer generation:

1. A prompt is evaluated by Jev or an LLM to select the appropriate tool.
2. The application chooses the execution path based on that decision.
3. A browser session opens to search, navigate, or retrieve content.
4. A language model summarizes the retrieved page or answers the original question.
5. Timing, token usage, and spend are logged and compared in the dashboard.

This separation keeps the decision layer measurable without conflating it with generative content creation.

## Repository structure

- [backend/](backend/) — FastAPI backend, benchmark logic, browser agent, and service modules
- [backend/app/](backend/app/) — application implementation and supporting services
- [frontend/](frontend/) — React + TypeScript dashboard and UI
- [data/](data/) — benchmark data and SQLite-backed results
- [.env.example](.env.example) — environment variable template
- [README.md](README.md) — project overview and setup instructions

## Technology stack

### Backend

- Python 3
- FastAPI
- SQLite
- Playwright
- httpx
- python-dotenv

### Frontend

- React 18
- TypeScript
- Vite
- Recharts

## Quick start

### 1. Configure environment

Create a local environment file and add your keys:

```bash
cp .env.example .env
```

Example configuration:

```bash
TYPESAFE_API_KEY=your-typesafe-key
OPENAI_API_KEY=your-openai-key
OPENAI_MODEL=gpt-4o-mini
JEV_IN_PER_M=0.042
DEMO_BUDGET_USD=0.5
BROWSER_HEADED=1
```

Notes:

- keep secrets in the local .env file only
- if keys are absent, the app clearly falls back to simulation mode
- setting BROWSER_HEADED=1 opens a visible browser; set it to 0 for headless execution

### 2. Install backend dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 3. Install frontend dependencies

```bash
cd frontend
npm install
```

### 4. Install Playwright Chromium

```bash
cd backend
playwright install chromium
```

If this step is skipped, the browser workflow falls back to HTTP-based fetching instead of a real browser session.

## Run locally

Start the backend:

```bash
cd backend
uvicorn main:app --reload
```

Start the frontend:

```bash
cd frontend
npm run dev
```

The frontend connects to the backend at http://localhost:8000.

## Live workflow modes

The application exposes several live agent modes:

- LLM Only: the model chooses the tool and the app executes the task
- Jev + LLM: Jev selects the tool, then the browser agent performs the action and the LLM answers from the page
- Compare: Jev + LLM and LLM-only workflows run side by side and stream results for immediate comparison

Example prompts include:

- open www.wikipedia.org and search for NBA
- search for quantum computing in wikipedia.org
- Retrieve the page https://example.com/docs
- What is the latest news about Android 16?

The system also auto-corrects common typos such as wikipidea.org -> wikipedia.org.

## Benchmarking

The benchmark uses a fixed set of labeled tool-selection prompts. Each result captures:

- selected tool
- expected tool
- latency
- token counts
- estimated cost
- correctness
- model metadata

The dashboard aggregates this information to report:

- accuracy by model
- average latency by model
- total spend by model
- comparative savings

## Cost and pricing model

The pricing layer is configurable and designed to be easy to adjust without modifying the frontend.

Jev pricing differs from a standard LLM:

- input tokens are billed
- output tokens are free
- the configured Jev rate is applied to the input-token count only

This reflects the product model of a typed decision system rather than a generative model.

## API overview

### Health and status

- GET /health
- GET /api/status

### Decision endpoints

- POST /api/decision/jev
- POST /api/decision/openai
- GET /api/tools

### Agent endpoints

- POST /api/agent/llm-only
- POST /api/agent/browser
- POST /api/agent/llm/stream
- POST /api/agent/browser/stream
- POST /api/agent/compare/stream

### Benchmark endpoints

- POST /api/benchmark/run
- POST /api/benchmark/run-all
- GET /api/benchmark/results
- DELETE /api/benchmark/results
- GET /api/benchmark/summary
- GET /api/benchmark/cost-analysis
- GET /api/benchmark/latency-analysis

## Important notes and limitations

- the project is designed for local development and demos rather than cloud deployment
- the benchmark focuses on tool selection rather than arbitrary enterprise automation workflows
- cost estimates are approximate, particularly when providers do not return full pricing metadata
- a single benchmark run should be treated as directional, not definitive
- Google search is intentionally not a stable target for automation because it frequently triggers anti-bot protections

## Suggested workflow

1. Start the backend and frontend.
2. Enter a prompt in one of the live agent tabs.
3. Compare Jev + LLM and LLM-only behavior side by side.
4. Run the benchmark dashboard to evaluate quality across labeled examples.
5. Review latency and cost trends before deciding how the decision layer should be used in a broader system.

## License

This project is intended for research, benchmarking, and local experimentation. Use it in accordance with the relevant repository and dependency licenses.
