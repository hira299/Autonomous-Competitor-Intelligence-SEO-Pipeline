# 2. 🔍 Competitor SEO Agent

> **File:** `[PROD]_Competitor_SEO_Agent.json`  
> **Stack:** n8n · Google Gemini · ClickUp · PostgreSQL  
> **Loom:** https://www.loom.com/share/bec593a6caed4e4eb808a0bafdfa79a7

## The Problem
Manual competitor research was a recurring time sink. Identifying content gaps required reading competitor blogs, cross-referencing keyword data, writing briefs, and creating ClickUp tasks — all done manually every week.

## The Solution
A weekly autonomous SEO agent that scrapes a competitor's website, feeds the content to a Gemini AI strategist, extracts structured content gap opportunities, and automatically creates detailed ClickUp tasks with AI-generated SEO briefs attached.

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


## Key Engineering Decisions
- **JSON-mode enforcement** via system prompt — prevents Gemini from returning unstructured text
- **Defensive AI output parser** — handles all edge cases before downstream nodes receive data
- **Dual error path** — scraper failures are logged, not silently dropped
- **Brief attached as comment** — writers get context directly in ClickUp without switching tools
