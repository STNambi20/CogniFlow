# CogniFlow Research Agent

A small, inspectable research agent for answering research-style questions. It runs locally, uses Gemini Google Search grounding plus source-page extraction, shows its plan and tool trace, and returns findings linked to sources. It uses only Python's standard library.

## Run it

Python 3.10 or newer is recommended. No package installation is needed.

```powershell
cd C:\Users\sthir\OneDrive\Documents\Desktop\CogniFlow
.\venv\Scripts\Activate.ps1
python server.py
```

Open <http://127.0.0.1:8004>. The app owner sets the API key in the local `.env` file; research users never enter or receive it. `.env` is excluded by `.gitignore`. After editing it, restart `server.py` so the server loads the value. Stop the server with Ctrl+C.

Before the first run, open `.env` and set `GEMINI_API_KEY=your_real_key`. Keep the real value only in `.env`; `.env.example` is a safe template and must remain blank. The key is held by the server process and sent only in the request header to Google's Gemini API. It is not returned by the health endpoint, included in research responses, logged, or sent to the browser. `.gitignore` prevents accidental commits; it does not encrypt the local file.

Research requires a server-side Gemini API key with access to the selected model and Interactions API. Each research run uses Gemini Google Search grounding and a separate structured synthesis call; model usage may be billed and quota limits apply. Configure `GEMINI_MODEL` in `.env` if you want to choose another model. Authentication, model, quota, network, search, and extraction failures are reported clearly. Page extraction needs access to selected public pages; private and loopback destinations are blocked.

## Architecture

```text
Browser UI ── question only ──► localhost server ──► Gemini API
                                      │                  │
                    planner ─────────┘                  ├─ Tool 1: hosted web search
                    source filter ◄── cited URLs ───────┘
                         │
                    Tool 2: safe public page fetch + text extraction
                         │
                    Tool 3: grounded structured synthesis
                         ▼
                    answer + linked source IDs + confidence + limitations
```

`server.py` contains the HTTP handler, query-sensitive planner, Gemini Interactions API integration, page-fetch tool, relevance filtering, citation validation, and lightweight logs. `index.html` is a dependency-free browser UI that renders the answer and visible run trace. The planner adapts its strategy for comparisons, lists, time-sensitive questions, and general research. Google Search grounding returns cited URLs; the agent filters duplicate and weak candidates, fetches up to six public pages in parallel, and keeps cited search evidence when a page blocks extraction. The synthesis call receives the evidence memo and selected page text, and returned source IDs are checked before display.

## Output contract

Each run returns the user's question, short answer, key findings with source IDs, source titles and URLs, confidence plus its basis, assumptions, limitations, suggested next steps, synthesis mode, an orchestration trace, and elapsed time. The UI links each finding to the cited source card. HTTP errors cover empty, malformed, and oversized requests.

## Design decisions and tradeoffs

- A custom single-agent orchestrator keeps the plan and failure paths easy to inspect; a framework would add setup and hide little useful complexity for this scope.
- Search plus page extraction are separate tools: snippets discover leads, while fetched page text provides stronger evidence where available.
- Heuristic relevance scoring is deterministic and cheap, but it is less capable than semantic retrieval. Search results and page content can vary.
- The API key is configured by the app owner in a gitignored `.env` and read only by the server. There is no key input or key-writing API in the browser app. The server binds to loopback and checks local request origins.
- Confidence is deliberately capped at Medium until a real evaluation set supports calibration. Fewer than two usable sources produces Low confidence.
- Per-request network timeouts, one bounded retry for transient API errors, bounded response sizes, malformed-output handling, page-fetch fallback, structured logs, and source-ID validation cover basic reliability. This prototype has no account auth, rate limiting, caching, background jobs, or production observability.
- Search snippets and source text are treated as untrusted evidence. The synthesis instructions reject instructions embedded in source text. Page fetching blocks private IPs and validates redirect targets; DNS rebinding protection and broader production egress controls remain follow-up work.
- Google Search grounding and synthesis improve answer quality but add latency and API cost; the selected model and search context size are visible tradeoffs.

## Evaluation and demo

See [EVALUATION.md](EVALUATION.md) for test queries and a run log template. See [SAMPLE_OUTPUTS.md](SAMPLE_OUTPUTS.md) for the output contract and steps to capture real runs. A practical 3–5 minute demo: connect the key, show the planner and tool trace, run a comparison query, open a cited source, then demonstrate a failure case (invalid key, quota, or blocked page) and explain recovery. Record actual behavior in the evaluation notes.

## Production next steps

Add a search provider with an explicit API contract, async bounded fan-out, retry/backoff for transient errors, stronger relevance and claim-level citation checks, an injection-resistant tool boundary, per-run cost/latency/source-quality metrics, and a regression evaluation set. Add a non-agentic baseline to test whether planning/tool selection improves answer quality enough to justify its latency and cost.
