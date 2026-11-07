# LectureLens — Project Brief

## Goal
Build a timestamp-aware AI learning assistant. Learners pause a lecture, ask questions, and receive evidence-grounded answers using only transcript content available up to the current timestamp plus explicitly permitted learning resources.

## Differentiator
Strict future-content filtering before retrieval, verifiable source citations, and honest abstention when the evidence does not support an answer.

## MVP features
- FastAPI backend and vanilla HTML/CSS/JavaScript interface.
- Ingestion for PDF, PPTX, TXT, Markdown, SRT and VTT.
- Lexical retrieval with timestamp filtering before scoring.
- Source labels for video timestamps, PDF pages and slide numbers.
- Optional Groq chat completions via `GROQ_API_KEY`; API key stays on the server.
- Optional local Ollama generation and extractive fallback.
- Automatic study planner based on exam date, available daily minutes, weak topics and completed topics.
- Optional `PLAN_WITH_GROQ=true` to refine plan action wording without allowing the model to change dates or allocations.
- Tests for timestamp leakage boundaries, document availability, unsupported questions and planner output.

## Run locally
```bash
cd backend
python -m venv .venv
# Windows: .venv\\Scripts\\Activate.ps1
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python main.py
```
Open `http://127.0.0.1:8000` and API docs at `http://127.0.0.1:8000/docs`.

## Configure Groq
Set `GROQ_API_KEY` in the backend environment and optionally set `GROQ_MODEL=openai/gpt-oss-20b`. Do not put secrets in frontend code or commit them. Verify current models and rate limits in the Groq Console docs.

## Known limitations
- Video player is a mock timeline; no actual video URL/player synchronization yet.
- Sample transcript and notes are synthetic demo content.
- In-memory resources and sessions are not durable or user-isolated.
- Planner task completion state is not persisted.
- Production deployment requires authentication, upload validation, rate limiting, persistent storage and privacy review.
