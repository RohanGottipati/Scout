# Scout

> Every idea has a history. Most of it is buried.

Scout researches the history of a hackathon idea before you build it. Type an idea and a pipeline of research agents digs up prior hackathon projects, repos, live products and failure signals, then presents the findings as a page-flipping notebook and turns them into a build plan.

## Features

- **Multi-agent research pipeline** — expands the idea into search angles, runs parallel web searches, and reconstructs a timeline of previous attempts (what was built, what happened, lessons) and the live competitive landscape.
- **Pattern and gap analysis** — identifies the recurring failure pattern ("turn sentence"), the remaining gap, and a timing signal, with research-quality stats (evidence count, distinct domains, missing evidence categories).
- **Cross-idea memory** — each result is stored as a Backboard memory so later searches can surface failure signals from semantically adjacent ideas.
- **Notebook UI** — results are rendered as a flip-book (turn.js) with an index of attempts and a page per attempt.
- **Build plan** — a builder agent proposes what to borrow from winners, what to avoid, an MVP scope, v1 features, moat, risks and next steps. Plan notes are kept in session storage.
- **Curated demo** — any idea containing "mental health journaling app" returns a built-in fixture (no API key needed).

## Tech Stack

- **Backend:** Node.js + Express (CommonJS), Node's built-in test runner
- **Frontend:** React 18, React Router 7, Vite 6, Tailwind CSS 3, jQuery + turn.js (vendored in `frontend/public/vendor/`)
- **AI / memory:** Backboard.io — LLM calls (default `openai` / `gpt-4o`), web search, and assistant memories

## Architecture

```
POST /api/search
  → Expander   (search angles)
  → Digger     (parallel web searches + evidence normalisation)
  → Historian + Landscape (parallel: past attempts / live competitors)
  → Pattern    (recurring failure, using Backboard memories of adjacent ideas)
  → Synthesis  (gap + timing)
  → result stored as a Backboard memory → JSON response

POST /api/build
  → Builder    (build plan from the search result)
```

Each stage has its own timeout and fallback, so a partial result is returned if a stage fails or the 180 s overall limit is reached.

## Project Structure

```
backend/
  server.js            # Express server and API routes
  agents/              # expander, digger, historian, landscape, pattern, synthesis, builder
  lib/
    backboard.js       # Backboard API client (assistant, threads, messages, memories)
    gemini.js          # searchAndReason / reasonOnly helpers (call Backboard) + JSON parsing
    prompts.js         # Agent prompts
    evidence.js        # Source normalisation and research-quality scoring
    demo-fixture.js    # Curated "mental health journaling app" result and build plan
  test/                # node:test unit tests
frontend/
  index.html           # Loads vendored jQuery/turn.js and the React app
  public/vendor/       # jquery.js, turn.js
  src/
    App.jsx            # Routes: /, /results, /products, /product/:idx, /build
    context/           # ResultsContext (API calls, session storage)
    pages/             # CoverPage, NotebookPage, BuildPlanPage
    components/        # BookFlip, LoadingState, PageNav
    styles/            # globals.css
HEADSTONE_implementation.md  # Original hackathon planning spec (historical; the code has since diverged)
```

## Prerequisites

- Node.js 18+ (the backend uses the global `fetch`)
- npm
- A Backboard.io API key for live research (optional for the demo idea)

## Setup

```bash
# 1. Environment variables (read by the backend from the repo root)
cp .env.example .env
# edit .env and set BACKBOARD_API_KEY

# 2. Install dependencies
cd backend && npm install
cd ../frontend && npm install
```

**Backboard promo code:** `HUSKYHACKS26`

## Running

In two terminals:

```bash
# Backend — http://localhost:3001
cd backend
npm start        # or: npm run dev (node --watch)

# Frontend — http://localhost:5173
cd frontend
npm run dev
```

Open http://localhost:5173 and try **"mental health journaling app"** for the curated demo.

## Environment Variables

Backend (`.env` in the repo root):

| Variable | Purpose |
|----------|---------|
| `BACKBOARD_API_KEY` | Backboard API key. Without it, live searches return empty results; the demo idea still works. |
| `BACKBOARD_URL` | Backboard API base URL (default `https://app.backboard.io/api`). |
| `BACKBOARD_ASSISTANT_ID` | Optional existing assistant ID. If unset, one is created and cached in `backend/cache/assistant_id.txt` (gitignored). |
| `BACKBOARD_LLM_PROVIDER` | LLM provider passed to Backboard (default `openai`). |
| `BACKBOARD_MODEL_NAME` | Model name passed to Backboard (default `gpt-4o`). |
| `PORT` | Backend port (default `3001`). |

Frontend (optional, e.g. `frontend/.env`):

| Variable | Purpose |
|----------|---------|
| `VITE_API_URL` | Backend base URL (default `http://localhost:3001`). |

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/search` | Body `{ "idea": string }`. Runs the research pipeline and returns timeline, competitors, turn sentence, gap, clock, sources and research quality. |
| POST | `/api/build` | Body: the search result fields (`idea`, `timeline`, `competitors`, `gap`, `clock`, `turn_sentence`, `research_quality`). Returns `{ idea, plan }`. |
| GET | `/api/graph/stats` | Number of stored ideas and unique failure patterns in Backboard memory. |
| GET | `/api/demo` | Returns the curated demo search result. |

## Scripts

| Location | Command | Description |
|----------|---------|-------------|
| `backend/` | `npm start` | Start the API server |
| `backend/` | `npm run dev` | Start with `node --watch` |
| `backend/` | `npm test` | Run unit tests (`node --test`) |
| `frontend/` | `npm run dev` | Vite dev server |
| `frontend/` | `npm run build` | Production build to `frontend/dist/` |
| `frontend/` | `npm run preview` | Preview the production build |
