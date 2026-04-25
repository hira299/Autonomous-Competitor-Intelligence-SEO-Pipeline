# Autonomous Competitor Intelligence & SEO Pipeline

Weekly autonomous pipeline that extracts competitor content, runs AI-powered gap analysis, and creates structured ClickUp tasks with SEO briefs attached — with zero manual input.

---

## Demo

[▶ Loom Walkthrough](https://www.loom.com/share/bec593a6caed4e4eb808a0bafdfa79a7)

---

## The Problem

Manual competitor research is a recurring time cost. Identifying content gaps requires reading competitor pages, cross-referencing keyword data, writing briefs, and creating project tasks — all done manually, every week.

---

## Architecture

```
[TRIGGER] Weekly_Cron
  → [GET] Fetch_Competitor_Site
      → [TRANSFORM] Extract_Text_Content     [WARN] Trigger_Proxy_Rotation
          → [AI] Gemini_SEO_Strategist              → [INSERT] Log_to_DB (DLQ)
              → [TRANSFORM] Clean_AI_Output
                  → [POST] Create_ClickUp_Tasks
                      → [POST] Attach_SEO_Brief
```

---

## Engineering Highlights

**JSON-mode enforcement**
The Gemini system prompt enforces structured JSON output. The AI cannot return unstructured text — the downstream parser receives a guaranteed schema on every execution.

**Defensive output parser**
A transform node validates and sanitises AI output before any downstream node receives it. Malformed responses, missing fields, and truncated JSON are all handled explicitly — not silently dropped.

**Dual error path**
Scraper failures (blocked requests, proxy exhaustion, 4xx/5xx) are routed to a DLQ and logged to PostgreSQL with the competitor URL, error type, and timestamp. The main pipeline continues for successful targets.

**Proxy rotation**
A parallel warning branch monitors scrape failure patterns and triggers proxy rotation before the scraper hits rate limits — reducing failed runs without manual intervention.

**Brief as comment**
SEO briefs are attached as ClickUp task comments, not as separate files. Writers see the full strategic context directly in the task view without switching tools.

---

## Node Breakdown

**[GET] Fetch_Competitor_Site**
HTTP request to the target competitor URL with configurable headers and proxy settings. Output: raw HTML.

**[TRANSFORM] Extract_Text_Content**
Strips HTML tags, navigation, footers, and boilerplate. Outputs clean body text for AI consumption. Reduces token count and noise in the LLM prompt.

**[AI] Gemini_SEO_Strategist**
System prompt instructs Gemini to act as an SEO strategist. Input: competitor content. Output: structured JSON array of content gap opportunities, each with title, target keyword, search intent, and recommended angle.

**[TRANSFORM] Clean_AI_Output**
Validates the JSON schema, handles edge cases (empty arrays, extra fields, truncated output), and formats each gap opportunity for ClickUp task creation.

**[POST] Create_ClickUp_Tasks**
Creates one ClickUp task per content gap opportunity in the configured list. Sets priority, due date, and custom fields from the AI output.

**[POST] Attach_SEO_Brief**
Posts the full AI-generated SEO brief as a comment on each newly created task.

**[INSERT] Log_to_DB (DLQ)**
Captures scraper failures with full error context in PostgreSQL. Enables replay and debugging without re-running the entire pipeline.

---

## Failure Handling

| Failure | Behaviour |
|---|---|
| Competitor site blocked | Proxy rotation triggered; failure logged to DLQ |
| Gemini returns malformed JSON | Parser catches error; task creation skipped; failure logged |
| ClickUp API rate limit | n8n retry with exponential backoff |
| Partial task creation | Already-created tasks are not duplicated; pipeline resumes from failure point |

---

## Setup

**Prerequisites**
- n8n instance (self-hosted)
- Google Gemini API key
- ClickUp API key and list ID
- PostgreSQL database (for DLQ)

**Steps**
```bash
# 1. Import workflow
# Import Competitor_SEO_Agent.json into n8n

# 2. Connect credentials
# Gemini API, ClickUp API, PostgreSQL

# 3. Configure targets
# Update the Fetch_Competitor_Site node with your target URLs

# 4. Configure ClickUp
# Set your list ID and custom field mappings

# 5. Run DLQ schema
# Execute /docs/dql_schema.sql in your PostgreSQL instance

# 6. Activate
```

**.env.example**
```
GEMINI_API_KEY=your_key_here
CLICKUP_API_KEY=your_key_here
CLICKUP_LIST_ID=your_list_id
PG_CONNECTION_STRING=postgresql://user:pass@host:5432/db
```

---

## Stack

- **Orchestration:** n8n
- **AI:** Google Gemini
- **Task Management:** ClickUp API
- **Persistence:** PostgreSQL
- **Scraping:** HTTP node with proxy rotation

---

## License

MIT
