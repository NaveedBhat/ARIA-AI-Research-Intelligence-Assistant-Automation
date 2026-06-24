<div align="center">

# ARIA
### AI Research Intelligence Assistant & Automation

*An autonomous AI system that aggregates, analyzes, and delivers curated AI intelligence — daily.*

![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-n8n-orange?style=flat-square)
![Primary AI](https://img.shields.io/badge/AI-Gemini%20Flash-blue?style=flat-square)
![Fallback AI](https://img.shields.io/badge/Fallback-Groq%20Llama%203.1-purple?style=flat-square)
![Language](https://img.shields.io/badge/Language-JavaScript-yellow?style=flat-square&logo=javascript)
![Database](https://img.shields.io/badge/Database-Google%20Sheets-34A853?style=flat-square&logo=google-sheets&logoColor=white)
![Delivery](https://img.shields.io/badge/Delivery-Telegram-2CA5E0?style=flat-square&logo=telegram&logoColor=white)
![Nodes](https://img.shields.io/badge/Nodes-42-red?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)
![Author](https://img.shields.io/badge/Author-Naveed%20Bhat-purple?style=flat-square)
![Last Updated](https://img.shields.io/badge/Last%20Updated-June%202026-informational?style=flat-square)

<br/>

![AI/ML](https://img.shields.io/badge/topic-AI%2FML-blueviolet?style=flat-square)
![Automation](https://img.shields.io/badge/topic-Automation-orange?style=flat-square)
![LLM](https://img.shields.io/badge/topic-LLM-blue?style=flat-square)
![News Aggregation](https://img.shields.io/badge/topic-News%20Aggregation-teal?style=flat-square)
![No Code](https://img.shields.io/badge/topic-No--Code-red?style=flat-square)
![RSS](https://img.shields.io/badge/topic-RSS-9c27b0?style=flat-square)
![Open Source](https://img.shields.io/badge/topic-Open%20Source-success?style=flat-square)

</div>

---

## Overview

ARIA is a fully automated AI workflow that monitors 14 premium AI news sources, analyzes articles using large language models, and delivers structured intelligence briefings — every single day without any human intervention.

It is designed to replace manual news aggregation with a zero-touch, always-on system that runs on a schedule, remembers what it has already seen, tracks trends over time, and delivers clean, curated content directly to your phone.

> **Built with reliability at its core** — ARIA includes a dual-AI fallback system that automatically switches from Gemini to Groq when the primary AI is unavailable, ensuring uninterrupted delivery. If both AIs fail, you receive an instant error alert with full diagnostics.

---

![ARIA Overview](ARIA/ARIA_01.png)
![ARIA Overview 2](ARIA/ARIA_02.png)

---

## How It Works

ARIA runs on two separate schedules:

### Daily Morning Brief (6 AM IST, every day)

```
14 RSS Feeds → Merge (~1,900 articles) → AI Filter + Dedup → LLM Analysis → Quality Gate → Storage + Delivery
```

### Weekly Digest (8 AM IST, every Sunday)

```
Read Sheets Database → Filter Last 7 Days → AI Summary → Telegram Weekly Report
```

| Stage | What Happens |
|-------|-------------|
| **Aggregation** | Reads ~1,900 articles across 14 curated AI news RSS feeds |
| **Filtering** | Keyword-based AI filtering reduces to ~50 relevant articles |
| **Deduplication** | Cross-run memory skips articles already seen in previous runs |
| **LLM Analysis** | Single batch call to Gemini Flash (or Groq if Gemini fails) |
| **Quality Gate** | Only articles scoring 7/10 or higher are saved |
| **Storage** | Structured metadata saved to Google Sheets database |
| **Trend Tracking** | Weekly statistics (categories, tags, scores) logged to Trends tab |
| **Delivery** | Formatted morning brief sent to Telegram |

---

## Key Features

- **Automated Daily Briefing** — Runs at 6 AM IST every day with zero manual intervention
- **AI-Powered Analysis** — Relevance scoring (1-10), category detection, sentiment analysis, and trend tagging per article
- **Dual-AI Redundancy** — Automatic failover from Gemini Flash to Groq Llama 3.1 when primary AI is unavailable
- **Cross-Run Deduplication** — Remembers every article URL across runs — never saves the same article twice
- **Weekly Digest** — Every Sunday, an AI-generated summary of the week's top 10 stories is delivered to Telegram
- **Trend Tracking** — Records category, tag, and score statistics each run; maintains an 8-week rolling window in the Trends sheet
- **Error Alerting** — If both AIs fail, an instant Telegram alert is sent and a full error record is logged to the Error Log sheet
- **Structured Database** — Three Google Sheets tabs: `Sheet1` (articles), `Trends` (weekly stats), `Error Log` (failures)
- **Telegram Delivery** — Clean, formatted Markdown brief with emoji formatting, article scores, and direct links

---

## Architecture

<div align="center">

```
                    ┌──────────────────────────────┐
                    │    Schedule Trigger (6AM IST) │
                    └─────────────┬────────────────┘
                                  │
                    ┌─────────────▼────────────────┐
                    │   14 RSS Feeds (parallel)     │
                    │   arxiv · google · deepmind   │
                    │   openai · huggingface · ...  │
                    └─────────────┬────────────────┘
                                  │  ~1,900 articles
                    ┌─────────────▼────────────────┐
                    │  Filter + Cross-Run Dedup     │
                    │  AI keywords → top 10 new     │
                    └─────────────┬────────────────┘
                                  │
               ┌──────────────────▼──────────────────┐
               │        Primary AI — Gemini Flash     │
               └──────┬──────────────────────┬───────┘
                      │ ✅ Success            │ ❌ Error
                      │                       ▼
                      │         ┌─────────────────────────┐
                      │         │  Fallback AI — Groq      │
                      │         │  Llama 3.1 8B Instant    │
                      │         └──────┬──────────┬───────┘
                      │                │ ✅        │ ❌
                      │                │          ▼
                      │                │  🚨 Format Error Alert
                      │                │  📋 Log Error to Sheet
                      │                │  📱 Send Error Alert
               ┌──────▼────────────────▼──────┐
               │     Parse + Quality Gate ≥ 7  │
               └──────────────┬───────────────┘
                              │
              ┌───────────────┴──────────────────┐
              ▼                                   ▼
  ┌─────────────────────┐            ┌──────────────────────┐
  │  Google Sheets       │            │  Update Trend Tracker │
  │  (Sheet1 database)   │            │  → Append Trends tab  │
  └────────────┬────────┘            └──────────────────────┘
               │
  ┌────────────▼────────┐
  │  Format Telegram     │
  │  → Send Morning Brief│
  └─────────────────────┘

  ─────────────────────────────────────────────────────
  WEEKLY (Sunday 8AM IST):
  Read Sheet1 → Filter 7 days → Gemini/Groq Summary
  → Format Weekly Digest → Send to Telegram
```

</div>

### n8n Workflow Canvas
![ARIA Workflow](ARIA/ARIA_FULL.png)

### Google Sheets Database Output
![ARIA Excel Output 1](ARIA/ARIA_EXCEL_01.png)
![ARIA Excel Output 2](ARIA/ARIA_EXCEL_02.png)
![ARIA Excel Output 3](ARIA/ARIA_EXCEL_03.png)

### Telegram Daily Brief
![ARIA Telegram Bot 1](ARIA/ARIA_BOT_01.png)
![ARIA Telegram Bot 2](ARIA/ARIA_BOT_02.png)
![ARIA Telegram Bot 3](ARIA/ARIA_BOT_03.png)

---

## Data Schema

### Sheet1 — Main Article Database

| Field | Description |
|-------|-------------|
| `ID` | Unique ARIA article identifier |
| `Date` | Processing date |
| `Title` | Article headline |
| `Link` | Source URL |
| `Summary` | AI-generated 3-sentence summary |
| `Category` | Detected topic category (e.g., AI Research, LLMs, GPU Programming) |
| `Relevance Score` | 1–10 AI relevance rating (only ≥ 7 saved) |
| `Sentiment` | Positive / Neutral / Negative |
| `Trend Tags` | Extracted topic hashtags (e.g., `#LLMs #Agents #OpenSource`) |
| `Key Entities` | Companies, models, and people mentioned |
| `Used In Newsletter` | Yes / No |

### Trends Tab — Weekly Statistics

| Field | Description |
|-------|-------------|
| `Week` | ISO week identifier (e.g., `2026-W26`) |
| `Total Articles` | Articles saved that run |
| `Avg Score` | Average relevance score |
| `Top Category` | Most common category this week |
| `Top Tag` | Most used trend tag this week |

### Error Log Tab — Failure Records

| Field | Description |
|-------|-------------|
| `Timestamp` | When the failure occurred (IST) |
| `Node Name` | Which n8n node failed |
| `Error Message` | Short description of the error |
| `Status` | Always `Failed` (for filtering) |

---

## Tech Stack

| Tool | Role | Free Tier |
|------|------|-----------|
| [n8n Cloud](https://n8n.io) | Workflow automation — 42-node pipeline | 1,000 executions/month |
| [Google Gemini Flash](https://aistudio.google.com) | Primary LLM — article analysis + weekly digest | 1,500 req/day |
| [Groq — Llama 3.1 8B](https://console.groq.com) | Fallback LLM — auto-activates on Gemini failure | 14,400 req/day |
| [Google Sheets](https://sheets.google.com) | Structured data persistence (3 tabs) | Unlimited |
| [Telegram Bot API](https://core.telegram.org/bots) | Daily + weekly brief delivery | Unlimited |

**Total running cost: $0/month**

---

## Reliability

| Scenario | Outcome |
|----------|---------|
| Gemini available | ✅ Full brief delivered via Gemini |
| Gemini rate limited | ✅ Full brief delivered via Groq (auto-fallback) |
| Gemini returns 503 | ✅ Full brief delivered via Groq (auto-fallback) |
| Both AIs unavailable | 🚨 Instant Telegram alert + Error Log row saved |
| Article already seen | ✅ Skipped by cross-run deduplication |
| Sunday trigger fires | 📰 Weekly AI-summary digest delivered via Telegram |

---

## Prerequisites

To run your own instance of ARIA, you will need:

- An [n8n Cloud](https://app.n8n.cloud) account or self-hosted n8n instance
- A [Google Gemini API key](https://aistudio.google.com/app/apikey) (free tier)
- A [Groq API key](https://console.groq.com) (free tier)
- A Telegram Bot token (via [@BotFather](https://t.me/BotFather))
- A Google Sheets spreadsheet with `Sheet1`, `Trends`, and `Error Log` tabs

---

## Configuration

All secrets are stored directly in n8n's built-in credential manager — never in code or configuration files.

| Secret | Where Used |
|--------|-----------|
| `Gemini API Key` | HTTP Request URL (`?key=...`) |
| `Groq API Key` | HTTP Request Authorization header |
| `Telegram Bot Token` | n8n Telegram credential |
| `Google Sheets OAuth2` | n8n Google Sheets credential |

> ⚠️ **Security Note:** Never expose API keys in public repositories. Use n8n's built-in credential system to store all secrets securely.

---

## Sources Monitored

| Source | Feed |
|--------|------|
| arXiv — AI Papers | `rss.arxiv.org/rss/cs.AI` |
| arXiv — ML Papers | `rss.arxiv.org/rss/cs.LG` |
| KDNuggets | `kdnuggets.com/feed` |
| Google AI Blog | `blog.google/technology/ai/rss/` |
| Simon Willison | `simonwillison.net/atom/everything/` |
| Microsoft Research | `microsoft.com/en-us/research/feed/` |
| OpenAI News | `openai.com/news/rss` |
| Towards Data Science | `towardsdatascience.com/rss` |
| VentureBeat | `venturebeat.com/feed/` |
| TechCrunch | `techcrunch.com/feed/` |
| MIT Technology Review | `technologyreview.com/feed/` |
| Google DeepMind | `deepmind.google/blog/rss` |
| HuggingFace Blog | `huggingface.co/blog/rss.xml` |
| The Hacker News | `feeds.feedburner.com/TheHackersNews` |

---

## License

This project is licensed under the [MIT License](LICENSE).

```
MIT License — Copyright (c) 2026 Naveed Bhat

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software to use, copy, modify, and distribute, subject to the
following conditions: The above copyright notice and this permission notice
shall be included in all copies or substantial portions of the Software.
```

---

## Author

**Naveed Bhat**

> *"The goal was not to read less — it was to read better."*

---

<div align="center">

*ARIA — Built June 2026 | 42-node n8n workflow | 14 RSS sources | Dual-AI redundancy*

</div>
