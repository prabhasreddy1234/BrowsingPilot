# Jev Decision Benchmark

A local benchmark and browser automation app that compares a TypeSafe Jev decision layer against a traditional LLM for tool selection and web-grounded task execution.

The project is designed around a simple principle:

> Jev makes the decision. The LLM writes the answer. Your code owns the control flow.

This allows the app to evaluate whether a typed, probabilistic decision model improves tool choice quality, latency, and cost while still using an LLM for natural-language reasoning and summarization.

## Overview

This repository contains:

- a Python FastAPI backend that runs decision benchmarks, stores results in SQLite, and streams agent activity over Server-Sent Events
- a React + TypeScript frontend that visualizes benchmark outcomes and exposes live agent workflows
- browser automation powered by Playwright Chromium
- a comparison layer that runs Jev + LLM and LLM-only pipelines side by side

The application is intended for local experimentation and demos rather than production deployment.

## What Jev is

Jev is TypeSafe's System One decision model. It returns a typed, probabilistic decision rather than a free-form text completion. In practice, that means:

- a selected tool
- a confidence score
- per-option probabilities
- low latency, typically in the ~70–200 ms range
- pricing based on input tokens only, with output free

By default, the project is configured for a Jev price of $0.042 per 1M input tokens and allows the app to run in simulation mode when no TypeSafe API key is available.

## Key features

- Benchmark dashboard comparing Jev vs LLM on labeled tool-selection queries
- Live cost, latency, and accuracy reporting
- SQLite-backed historical results and summary endpoints
- Browser-driven task execution for site navigation and search tasks
- Visible browser mode for interactive demos and headless comparison mode
- Streaming workflow events with cumulative timing, token usage, and spend
- Graceful browser fallback when a real browser cannot be launched
- Common typo correction for domains such as Wikipedia and Bing

## Architecture

The system separates decision-making from answer generation:

1. A user prompt is evaluated by Jev or an LLM to select a tool
2. The app decides the execution path based on the selected tool
3. A browser opens to fetch or search a page
4. A language model summarizes the page or answers the user query
5. Timing, tokens, and spend are logged and compared in the dashboard

This arrangement makes the decision layer easy to benchmark without conflating it with text generation.

## Repository layout

- [backend/](backend/) — FastAPI API, SQLite persistence, benchmark logic, browser agent, decision providers
- [backend/app/](backend/app/) — application code and service modules
- [frontend/](frontend/) — React + TypeScript dashboard and UI
- [data/](data/) — project data and benchmark database storage
- [.env.example](.env.example) — template for required environment variables
- [README.md](README.md) — project documentation

## Tech stack

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

Copy the example environment file and set your real keys in a local .env file:

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

- Keep keys in the local .env file only
- If no keys are present, the app falls back to clearly labeled simulation mode
- BROWSER_HEADED=1 opens a visible browser for the live browser tabs; set it to 0 for headless execution

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

### 4. Install Chromium for Playwright

```bash
cd backend
playwright install chromium
```

If this step is skipped, the browser workflow falls back to HTTP fetching instead of a real browser session.

## Run the app locally

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

The frontend expects the backend at http://localhost:8000.

## Browser workflows

The app exposes multiple live agent modes:

- LLM Only: the model chooses the tool and the app opens a browser to perform the action
- Jev + LLM: Jev selects the tool, then the browser runs and the LLM answers from the page
- Compare: Jev + LLM and LLM-only execute concurrently and stream their results side by side

Supported prompt patterns include:

- open www.wikipedia.org and search for NBA
- search for quantum computing in wikipedia.org
- Retrieve the page https://example.com/docs
- What is the latest news about Android 16?

The agent also auto-corrects common domain typos such as wikipidea.org -> wikipedia.org.

## Benchmarking

The benchmark is built around a fixed labeled dataset of tool-selection prompts. Each result records:

- selected tool
- expected tool
- latency
- token counts
- estimated cost
- correctness
- model metadata

The dashboard can aggregate results across runs and report:

- accuracy by model
- average latency by model
- total spend by model
- comparative savings

## Cost and pricing model

The pricing layer is configurable and is intended to be easy to adjust without changing the frontend.

Jev is billed differently from a standard LLM:

- input tokens are billed
- output tokens are free
- the configured Jev rate is applied to the input-token count only

This is implemented in the backend pricing and config modules and reflects the product model of a typed decision system rather than a generative model.

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

- This project is designed for local development and demos, not cloud deployment
- The benchmark focuses on tool selection, not full end-to-end automation across arbitrary enterprise workflows
- Cost estimates are approximate, especially when providers do not return full pricing metadata
- API outputs can vary over time, so a single benchmark run is best interpreted as directional rather than definitive
- Google search is intentionally not a reliable target for automation because it frequently triggers anti-bot protections

## Suggested workflow

1. Start the backend and frontend
2. Enter a prompt in one of the live agent tabs
3. Compare Jev + LLM vs LLM-only behavior
4. Run the benchmark dashboard to evaluate model quality over labeled examples
5. Review latency and cost trends before deciding how the decision layer should be used in a broader system

## License

This project is for research, benchmarking, and local experimentation. Use it in line with the relevant repository and dependency licenses.
