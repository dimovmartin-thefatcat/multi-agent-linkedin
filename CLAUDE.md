# multi_agent — LinkedIn Content Pipeline

## What This Project Does

A 3-agent pipeline that takes a free-form topic and a format choice, searches the web in real time, analyses the findings for a professional audience, and outputs a ready-to-post LinkedIn post or article — with hook, body, emojis, references, and hashtags.

## Environment

- Python 3.14.7 via `py` launcher
- Virtual environment: `venv/` (project root — VS Code auto-discovers it)
- Jupyter kernel: `"Python (venv)"` registered at `%APPDATA%\jupyter\kernels\venv`
- Key packages: `openai-agents`, `python-dotenv`, `ipykernel`, `pydantic`

## API Keys

Stored in `.env` (never commit this file):
```
OPENAI_API_KEY=...
TAVILY_API_KEY=...
```
Loaded via `load_dotenv()` at the top of the notebook.

## Project Files

| File | Purpose |
|---|---|
| `multi_agent_workflows.ipynb` | Main notebook — all agents, pipeline, and tests |
| `.env` | API keys (gitignored) |
| `.vscode/settings.json` | Points VS Code to `venv/Scripts/python.exe` |
| `venv/` | Isolated Python environment |

## Agent Architecture

```
user_query + format_type
       │
       ▼
  Researcher  ──── tavily_search tool ──── ResearchOutput.summary (5 bullets)
       │
       ▼
   Analyst    ──── no tools ──────────── ResearchOutput.summary (2 paragraphs)
       │
       ▼
    Writer    ──── no tools ──────────── LinkedInOutput
```

All three agents use `gpt-5-mini`. A shared `SQLiteSession(uuid)` preserves context across the run.

## Pydantic Models

```python
class ResearchOutput(BaseModel):
    summary: str                     # used by Researcher and Analyst

class LinkedInOutput(BaseModel):
    hook: str                        # opening line
    body: str                        # full post/article with emojis
    hashtags: List[str]              # 3-7 tags, never inside body
    format_used: str                 # traceability
```

## Format Types

| `format_type` | Length | Style |
|---|---|---|
| `"short_post"` | 150–300 w | Hook + bullets + CTA |
| `"long_article"` | 700–1200 w | Structured editorial, references |
| `"celebration"` | 100–200 w | Milestone, warm/personal |
| `"reaction"` | 150–250 w | Clear stance + closing question |

## How to Run

1. Fill in `.env` with real API keys.
2. Open `multi_agent_workflows.ipynb` in VS Code, select kernel **Python (venv)**.
3. Run all cells top to bottom.
4. In the last cell, set `user_query` and `format_type`, then run:

```python
post = await manager_run(user_query, format_type)
print(post.hook)
print(post.body)
print("  ".join(post.hashtags))
```

## Comment Agent

`comment_run(article_text, stance, article_url=None, word_count=80) -> CommentOutput` drafts a reply to someone else's post. The pipeline is Reader → Researcher → Commenter. The comment is a **draft**: post it by hand, because LinkedIn doesn't allow automated commenting from personal accounts.

- Stances: `add_value`, `new_perspective`, `question`, `congrats`
- Rules: 1–3 specific article points, always positive (never criticise), length set by `word_count` (±10%), no hashtags, written in the `voice_profile.md` voice
- `article_url` is optional. Tavily extract fails on login-walled LinkedIn pages, and then the pasted `article_text` is used
- `export_comment_md(reply)` saves the comment to `outputs/comment_<stance>_<timestamp>.md`

## Key Tool: `tavily_search`

Calls `https://api.tavily.com/search` directly via `requests`. Defined with `@function_tool` wrapping `_tavily_search_impl()`. Test the HTTP layer without the SDK by calling `_tavily_search_impl(query, max_results)` directly.

## Common Issues

| Error | Fix |
|---|---|
| `SQLiteSession.__init__() missing session_id` | Pass a string ID: `SQLiteSession(str(uuid.uuid4()))` |
| `coroutine object` printed | `on_invoke_tool` is async — use `await` |
| `python not found` | Use `py` launcher (Python 3.14 installed via Microsoft Store path) |
