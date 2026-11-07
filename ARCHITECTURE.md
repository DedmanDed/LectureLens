# Architecture

```text
Browser (vanilla HTML/JS)
  ├─ mock lecture timeline -> current_time (seconds)
  ├─ POST /api/ask {question, current_time, session_id}
  └─ POST /api/upload (PDF/PPTX/TXT/MD/SRT/VTT)
                  |
                  v
             FastAPI API
                  |
          parsers + chunk metadata
                  |
       in-memory chunk store (MVP)
                  |
     timestamp filter BEFORE ranking
       video.start <= current_time
       + permitted documents
                  |
        lexical retrieval (top 5)
                  |
      optional local Ollama generation
        /                  \
     available           unavailable
       |                     |
 grounded synthesis     extractive evidence
       \                     /
        answer + real source labels
                  |
                  v
          UI source citations
```

**Trust boundary:** The client sends the current timestamp, so a production video player must have the server verify the session/video identity and use a trusted playback position or signed progress token if anti-cheating matters. For the hackathon prototype, the server applies the timestamp filter to every query, but the client timestamp itself is not authenticated.
