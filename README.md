# LectureLens — timestamp-aware RAG MVP

LectureLens is a working local MVP for the AHGV Buildverse PS-01 brief. It implements the core watch/pause/ask experience with server-side timestamp filtering, transcript/document ingestion, retrieval, source citations, and a grounded extractive fallback. Groq API integration can synthesize grounded explanations; without a key, the app uses a safe extractive fallback. A built-in study planner creates a deadline-aware revision schedule and can optionally use Groq to improve task guidance.

## Stack and why

- **Python + FastAPI**: small, documented HTTP API with interactive OpenAPI docs.
- **In-memory chunk store + lexical TF-IDF-like scoring**: zero-cost, no vector database to operate for the first demo. Replace with hybrid/vector retrieval when the dataset grows.
- **pypdf + python-pptx**: PDF page and PowerPoint slide citations.
- **Groq API (optional)**: backend-only hosted inference using the OpenAI-compatible Groq endpoint. If no key is configured or the API fails, the app quotes retrieved evidence rather than inventing a generated answer. Optional Ollama fallback remains available.
- **Automatic study planner**: schedules topics around an exam date, daily study time, weak topics and completed topics. The deterministic planner works without any LLM key; optional Groq enhancement is opt-in.
- **Vanilla HTML/CSS/JS**: reuses the starter UI and avoids a frontend build step.

## Run locally

Use Python 3.10+.

```bash
cd backend
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

Open <http://127.0.0.1:8000>. API docs: <http://127.0.0.1:8000/docs>. Health check: <http://127.0.0.1:8000/api/health>.

The app seeds a sample timestamped lecture and sample notes. Upload `.pdf`, `.pptx`, `.txt`, `.md`, `.srt`, or `.vtt` using **Add learning material**. For transcript-aware filtering, upload SRT/VTT or plain text with timestamps such as `[00:12:34] transcript segment`. Untimed text is treated as a document, not as video.

> The player is still a mock timeline, not a real video playback integration. The sample transcript is synthetic test content, not a transcript of the titled paper/video. Uploaded files and sessions are stored in process memory and disappear on restart. Legacy binary `.ppt` is not supported; convert it to `.pptx`.

## Groq API (recommended hosted generation)

1. Create an API key at <https://console.groq.com/keys>.
2. Copy `.env.example` to `.env` for your own reference, but note that this MVP does not auto-load `.env`; export variables in your shell or configure them in your hosting provider's secret settings.
3. Set `GROQ_API_KEY` and optionally `GROQ_MODEL` (default: `openai/gpt-oss-20b`). Check Groq's current model list and rate limits at <https://console.groq.com/docs/models> and <https://console.groq.com/docs/rate-limits>.

Windows PowerShell:

```powershell
$env:GROQ_API_KEY="your-key-here"
$env:GROQ_MODEL="openai/gpt-oss-20b"
python main.py
```

macOS/Linux:

```bash
export GROQ_API_KEY="your-key-here"
export GROQ_MODEL="openai/gpt-oss-20b"
python main.py
```

Never put a real API key in frontend JavaScript, commit it to Git, or share it in screenshots. The key is used only by the backend. If the key is absent, rate-limited or unavailable, the assistant falls back to evidence excerpts. To optionally allow Groq to refine study-plan task wording, set `PLAN_WITH_GROQ=true`; the dates and minute allocations are created by the deterministic planner first.

## Automatic study planner

Open **Study planner** in the app. Enter an exam date, daily study minutes, topics, weak topics and already completed topics. The `POST /api/plan` endpoint returns a dated plan and task actions. The rules-based planner works without Groq and schedules light revision on the exam date. Planner checkboxes are currently session-only and are not saved across reloads.

## Optional free local LLM

Install [Ollama](https://ollama.com), then:

```bash
ollama pull qwen2.5:3b
```

Copy `.env.example` values into your shell or configure them in your deployment environment:

```bash
# Windows PowerShell
$env:USE_OLLAMA="true"
$env:OLLAMA_MODEL="qwen2.5:3b"
python main.py
```

```bash
# macOS/Linux
export USE_OLLAMA=true
export OLLAMA_MODEL=qwen2.5:3b
python main.py
```

When Ollama is enabled but unavailable, the API falls back to extractive evidence mode. No paid LLM API key is required for the local fallback.

## API

### `POST /api/ask`

```json
{
  "question": "What does self-attention do?",
  "current_time": 1122,
  "session_id": "demo-session"
}
```

Response includes `answer`, `sources` (video timestamps, document pages or slide numbers), `retrieved_count`, `generation_mode`, and `latency_ms`.

### `POST /api/upload`

Multipart form upload; repeat the `files` field for multiple resources. Max size: 15 MB per file.

### `POST /api/plan`

Accepts `exam_date` (`YYYY-MM-DD`), `daily_minutes`, `topics`, `weak_topics`, and `completed_topics`. Returns a day-by-day study schedule.

### Other endpoints

- `GET /api/health`
- `GET /api/resources`
- `POST /api/reset` (restores sample data and clears in-memory uploads)

## Timestamp boundary design

The critical rule is enforced in `retrieve()` **before scoring**. Video chunks whose end timestamp is later than the playhead are excluded entirely, including the currently incomplete cue, to avoid leaking the rest of a sentence:

```python
eligible = [c for c in CHUNKS if c["kind"] != "video" or c["end"] <= current_time]
```

Only then are the eligible chunks scored. Supporting documents are explicitly allowed by the challenge, so they can explain a topic that has not yet appeared in the video. Source labels distinguish permitted documents from lecture timestamps. If no evidence is retrieved, the fallback says it cannot verify the answer.

## Test

```bash
python -m unittest discover -s tests -v
```

Tests check that future video segments are excluded, permitted supporting documents remain searchable, and unrelated questions do not produce hits.

## Known limitations before judging

1. **Not yet hosted**: this is a local MVP; the challenge requires a public deployment. Deploy the FastAPI app to a suitable host and verify file persistence/security before submitting.
2. **No real video player or automatic transcription**: timeline is a mock control; transcript must be supplied as VTT/SRT/timestamped text.
3. **No durable or per-user storage/auth**: the chunk store and chat history are in memory and shared by this single-process demo. Do not expose it publicly as-is.
4. **Retrieval is lexical, not semantic embeddings**: use hybrid lexical + embedding search and a persistent vector store after the end-to-end flow is stable.
5. **PPTX supported; legacy PPT not supported** despite its file extension being allowed. Convert legacy files to PPTX first.
6. **Groq generation requires a valid API key and available model/rate limit**. If unavailable, responses are evidence excerpts, not an LLM-written explanation. Ollama remains an optional local fallback.
7. **PDF parsing is text-based**; scanned/image-only PDFs need OCR.
8. **Citation verification is structural**: citations are generated from actual retrieved chunks, but semantic claim-by-claim verification still needs an evaluation harness.

## Suggested 3–4 person split

- **Frontend/product**: real video player, playback timestamp events, transcript UX and citation jump.
- **Backend/retrieval**: parsers, chunk metadata, hybrid search, persistence and session isolation.
- **AI/evaluation**: local model prompt, abstention tests, future-content leakage tests, citation precision.
- **Deployment/QA** (if four people): containerize, deploy, load test, security checks and demo recording.

## Security notes

This is a hackathon MVP. Add authentication, user-scoped storage, upload malware/content limits, rate limiting, secure deployment settings and privacy controls before public use. Uploaded documents are treated as untrusted data, not instructions. Do not commit credentials.
