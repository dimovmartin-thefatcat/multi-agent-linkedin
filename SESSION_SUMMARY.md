# Session Summary — multi_agent LinkedIn Pipeline

## Objective

Build an automated, multi-agent pipeline that turns a free-form topic into a
ready-to-post LinkedIn piece — researched, analysed, and written in a chosen
format — using the OpenAI Agents SDK.

## What We Did

### 1. Environment setup
- Created `venv/` (Python 3.14.7 via `py` launcher) in the project root
- Installed `openai-agents`, `python-dotenv`, `ipykernel`, `pydantic`
- Registered Jupyter kernel **"Python (venv)"**; pointed VS Code at the venv
- Created `.env` for `OPENAI_API_KEY` and `TAVILY_API_KEY`

### 2. Tavily web-search tool
- Built `tavily_search` (`@function_tool`) wrapping a direct call to
  `https://api.tavily.com/search`, with a TypedDict input and a testable
  `_tavily_search_impl()` helper

### 3. Multi-agent pipeline
- **Researcher** — uses `tavily_search`, returns 5 bullet findings
- **Analyst** — extracts professional-audience insights (no tools)
- **Writer** — produces the final output (no tools)
- `manager_run()` chains all three over a shared `SQLiteSession`

### 4. LinkedIn transformation
- Swapped the generic report model for `LinkedInOutput`
  (`hook`, `body`, `hashtags`, `format_used`)
- Added format-aware Writer instructions for 4 formats:
  `short_post`, `long_article`, `celebration`, `reaction`
- `manager_run(user_query, format_type)` now drives the format choice

### 5. Markdown export
- Added `export_to_md()` to save any run as a formatted `.md` file with
  emojis and hashtags (generated files gitignored)

### 6. Docs & version control
- Wrote `CLAUDE.md` (project guide)
- Initialised git; pushed to GitHub:
  **https://github.com/dimovmartin-thefatcat/multi-agent-linkedin**
- Two commits: initial setup, then pipeline + export

### 7. Next step (planned, not built)
- Drafted `voice_profile.md` skeleton to capture personal tone, phrases,
  and example posts — to be loaded into the Writer agent

## Issues Resolved Along the Way
- `python not found` → used the `py` launcher
- Coroutine printed → `on_invoke_tool` is async, needs `await`
- `SQLiteSession` missing `session_id` → pass a UUID string
- API key 401 (expired) → replace key in `.env`, restart kernel
