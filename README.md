# LLM-Powered Code Review Bot

A GitHub App that automatically reviews pull requests using Claude/GPT-4, posting structured comments on code quality, bugs, security issues, and performance problems.

## Architecture

```
GitHub PR opened/updated
        ↓
GitHub Webhook → FastAPI endpoint
        ↓
Diff Parser (extract changed files + hunks)
        ↓
LLM Reviewer (Claude claude-sonnet-4-20250514 / GPT-4)
        ↓
Comment Formatter
        ↓
GitHub API → Post review comments on specific lines
```

## Key Design Decisions

| Decision | Choice | Reason |
|---|---|---|
| Review scope | Changed lines only (not full file) | Focused, relevant feedback |
| Model | Claude claude-sonnet-4-20250514 | Best code understanding, long context |
| Chunking | Per-file, max 4K tokens | Stays within context window |
| Comment style | Inline on specific lines | More actionable than summary comments |
| Rate limiting | 1 review per commit | Avoid spamming on rapid pushes |
| Secrets check | Regex pre-filter | Catch API keys before LLM call |

## Review Categories

- 🐛 **Bugs**: off-by-one errors, null pointer risks, race conditions
- 🔒 **Security**: SQL injection, hardcoded secrets, SSRF vulnerabilities
- ⚡ **Performance**: N+1 queries, unnecessary loops, missing indexes
- 📖 **Readability**: complex logic needing simplification, missing docstrings
- ✅ **Positive**: explicitly acknowledge good patterns

## Running Locally

```bash
pip install -r requirements.txt
# Set env vars: GITHUB_APP_ID, GITHUB_PRIVATE_KEY, ANTHROPIC_API_KEY
ngrok http 8000   # expose localhost for GitHub webhooks
uvicorn app.main:app --reload
```

## What to implement next

- [ ] Language-specific rules (Python type hints, Java null safety)
- [ ] Learning from accepted/dismissed suggestions
- [ ] PR summary comment with overall quality score
- [ ] Block merge if critical security issues found

## Interview Talking Points

**"How do you handle PRs with 100+ changed files?"**
Process files in parallel (asyncio.gather), prioritize by change size and file type (skip auto-generated, lock files, assets). Cap at 20 files per review to control cost and latency.

**"How do you prevent the bot from being annoying?"**
Only comment on lines with HIGH confidence issues. Batch all comments into a single review (not individual comments). Never comment the same issue twice across commits. Let users configure which categories they care about.

**"What's your prompt engineering approach?"**
Few-shot examples of good vs bad reviews. System prompt with explicit output format (JSON with line number, severity, category, suggestion). Temperature=0 for determinism. Ask the model to explain WHY something is a problem, not just that it is.
