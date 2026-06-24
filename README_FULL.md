# ARIA — AI Research Intelligence Assistant

> **Developed by Naveed Bhat**  
> An autonomous AI-powered news intelligence system that monitors, analyzes, and delivers daily AI news briefings via Telegram — with a built-in AI fallback system for 100% uptime.

---

## 📌 What is ARIA?

ARIA (AI Research Intelligence Assistant) is a fully automated n8n workflow that:

1. **Collects** AI news from 14 RSS feeds across the internet every day
2. **Filters** articles using AI keyword matching
3. **Analyzes** each article using Google Gemini AI (relevance score, summary, category, sentiment, tags)
4. **Stores** structured data in a Google Sheet database
5. **Falls back to Groq AI** automatically if Gemini is unavailable (rate limit/outage)
5. **Delivers** a formatted morning brief to your Telegram every day

ARIA runs automatically on a schedule — zero manual work required after setup.

---

## 🏗️ System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                          ARIA PIPELINE                               │
│                                                                      │
│  ┌──────────┐    ┌─────────────┐    ┌──────────┐                   │
│  │ Schedule │───▶│  14 RSS     │───▶│  Merge   │                   │
│  │ Trigger  │    │  Feed Nodes │    │ (Combine)│                   │
│  └──────────┘    └─────────────┘    └────┬─────┘                   │
│                                          │                           │
│                                          ▼                           │
│                                   ┌──────────────┐                  │
│                                   │ Code Node    │                  │
│                                   │ (AI Filter + │                  │
│                                   │  Dedup +     │                  │
│                                   │  Store data) │                  │
│                                   └──────┬───────┘                  │
│                                          │ 10 articles               │
│                                          ▼                           │
│                                   ┌──────────────┐                  │
│                                   │ HTTP Request │                  │
│                              ┌────│ (Gemini AI)  │────┐            │
│                              │✅  └──────────────┘  ❌│            │
│                         (success)                  (error)          │
│                              │                        │             │
│                              ▼                        ▼             │
│                       ┌──────────┐            ┌─────────────┐      │
│                       │  Code1   │            │  Groq Setup │      │
│                       │ (Parse)  │            │  (Code)     │      │
│                       └────┬─────┘            └──────┬──────┘      │
│                            │                         │              │
│                            │                         ▼              │
│                            │                  ┌─────────────┐      │
│                            │                  │ HTTP Request│      │
│                            │                  │ (Groq AI)   │      │
│                            │                  └──────┬──────┘      │
│                            │                         │              │
│                            │                         ▼              │
│                            │                  ┌─────────────┐      │
│                            │◀─────────────────│ Normalize   │      │
│                            │                  │ Groq        │      │
│                       ┌────┴─────┐            └─────────────┘      │
│                       │  Filter  │                                  │
│                       │ (≥ 7)    │                                  │
│                       └────┬─────┘                                  │
│                            │                                        │
│                   ┌────────┴──────────┐                            │
│                   ▼                   ▼                            │
│           ┌─────────────┐    ┌──────────────┐                     │
│           │ Google Sheet│    │Format Telegram│                     │
│           │ (Save)      │    │ + Send        │                     │
│           └─────────────┘    └──────────────┘                     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Tech Stack

| Tool | Purpose | Cost |
|------|---------|------|
| **n8n Cloud** | Workflow automation platform | Free tier |
| **Google Gemini 2.0 Flash** | Primary AI article analysis | Free tier |
| **Groq (Llama 3.1)** | Fallback AI (auto-activates if Gemini fails) | Free tier |
| **Google Sheets** | Article database storage | Free |
| **Telegram Bot API** | Morning brief delivery | Free |
| **14 RSS Feeds** | News data sources | Free |

**Total cost: $0/month** ✅

---

## 📡 Data Sources (14 RSS Feeds)

| Node | Feed | Source |
|------|------|--------|
| RSS Read | ArXiv CS.AI | arxiv.org/rss/cs.AI |
| RSS Read1 | ArXiv CS.LG | arxiv.org/rss/cs.LG |
| RSS Read2 | KDNuggets | kdnuggets.com/feed |
| RSS Read3 | Google AI Blog | blog.google/technology/ai/rss |
| RSS Read4 | RSS Read4 | (AI source) |
| RSS Read5 | Microsoft Research | microsoft.com/en-us/research/feed |
| RSS Read6 | OpenAI Blog | openai.com/news/rss |
| RSS Read7 | Towards Data Science | towardsdatascience.com/rss |
| RSS Read8 | VentureBeat AI | venturebeat.com/feed |
| RSS Read9 | TechCrunch AI | techcrunch.com/feed |
| RSS Read10 | MIT Tech Review | technologyreview.com/feed |
| RSS Read11 | DeepMind Blog | deepmind.google/blog/rss |
| RSS Read12 | HuggingFace Blog | huggingface.co/blog/rss.xml |
| RSS Read13 | Simon Willison | simonwillison.net/atom/everything |

---

## 🧩 Node-by-Node Breakdown

### 1. Schedule Trigger
- **Type:** n8n Schedule Trigger
- **Runs:** Twice daily (configurable — e.g., 6 AM + 6 PM IST)
- **Purpose:** Kicks off the entire pipeline automatically

---

### 2. RSS Feed Nodes (×14)
- **Type:** RSS Read node
- **What it does:** Fetches latest articles from each feed
- **Output fields:** `title`, `link`, `contentSnippet`, `pubDate`, `creator`, `content`, `categories`
- **Architecture:** 14 parallel nodes → grouped into 3 Merge nodes (n8n max input limit = 10)

---

### 3. Merge Nodes (Merge + Merge1 → Merge2)
- **Type:** Merge node (Append mode)
- **Why 3 nodes:** n8n Merge node accepts max 10 inputs. With 14 feeds, we use:
  - `Merge` — first 7 feeds
  - `Merge1` — next 7 feeds
  - `Merge2` — combines Merge + Merge1 output (~1,900+ articles total)

---

### 4. Code in JavaScript (Filter + Dedup)
- **Type:** Code node — Run Once for All Items
- **Input:** ~1,900+ raw RSS articles
- **What it does:**
  1. **AI Keyword Filter** — keeps only articles mentioning AI topics
  2. **Deduplication** — removes duplicate URLs within the batch
  3. **Batch Limiter** — caps at 10 articles to save API quota

**Keywords used:**
```
ai, artificial intelligence, machine learning, llm, gpt, gemini,
claude, model, neural, deep learning, openai, deepmind, anthropic,
automation, chatbot, transformer, agent, benchmark, diffusion,
nvidia, sam altman, hugging face, open source
```

- **Output:** 10 clean, unique, AI-relevant articles + pre-built `geminiBody` JSON

---

### 5. HTTP Request (Gemini AI Analysis)
- **Type:** HTTP Request node
- **Method:** POST
- **URL:** `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=YOUR_KEY`
- **Body Type:** Raw JSON
- **Body:** `{{ $json.geminiBody }}` (pre-built batch prompt)

**How it works — BATCH mode:**
Instead of calling Gemini 10 times (once per article), we send ALL 10 articles in ONE API call. This:
- Uses 1 API request instead of 10
- Avoids rate limiting
- Gets all analyses in ~30-60 seconds

**The Gemini Prompt:**
```
Analyze these 10 AI news articles. Return ONLY a JSON array.

1. Title: [article 1 title]
   Summary: [article 1 snippet]
   Link: [url]
...

Return: [{"index":1,"title":"...","link":"...","relevance_score":8,
"summary":"3 sentences","category":"AI Research","sentiment":"Positive",
"trend_tags":"#AI #LLMs","key_entities":"OpenAI, GPT"}]
```

---

### 6. Code in JavaScript1 (Parse Gemini Response)
- **Type:** Code node — Run Once for All Items
- **Input:** 1 item (Gemini's full response)
- **What it does:**
  1. Extracts the text from `candidates[0].content.parts[0].text`
  2. Strips markdown code fences (` ```json `)
  3. Parses the JSON array
  4. Maps each analysis to a formatted row with all 12 columns
  5. Generates unique ARIA ID for each article

**Output per article:**
```javascript
{
  ID: 'ARIA-1782229123-abc12',
  Date: '2026-06-23',
  Source: 'https://...',
  Title: 'Article Title',
  Link: 'https://...',
  Summary: 'Three sentence AI-generated summary...',
  Category: 'AI Research',
  'Relevance Score': 9,
  Sentiment: 'Positive',
  'Trend Tags': '#LLMs #OpenSource',
  'Key Entities': 'OpenAI, GPT-4',
  'Used In Newsletter': 'No'
}
```

---

### 7. Filter Node (Quality Gate)
- **Type:** Filter node
- **Condition:** `Relevance Score >= 7`
- **Purpose:** Only high-quality, truly AI-relevant articles pass through
- **Typical result:** 7-10 articles pass out of 10

---

### 8. Append or Update Row in Sheet (Google Sheets)
- **Type:** Google Sheets node
- **Operation:** Append or Update
- **Sheet:** ARIA — AI News Database
- **Match Column:** ID (prevents true duplicates)
- **Saves 12 columns:**

| Column | Description |
|--------|-------------|
| A — ID | Unique ARIA identifier |
| B — Date | Publication date |
| C — Source | Article domain |
| D — Title | Article headline |
| E — Link | Full URL |
| F — Summary | AI-generated 3-sentence summary |
| G — Category | AI Research / Industry News / Open Source / etc. |
| H — Relevance Score | 1-10 AI relevance rating |
| I — Sentiment | Positive / Negative / Neutral |
| J — Trend Tags | Hashtags (#LLMs #OpenAI etc.) |
| K — Key Entities | Named entities (companies, people, models) |
| L — Used In Newsletter | Yes / No (for Telegram tracking) |

---

### 9. Format Telegram (Code Node)
- **Type:** Code node — Run Once for All Items
- **Input:** Saved article rows
- **What it does:** Builds a beautifully formatted Telegram message

**Output message format:**
```
🤖 ARIA Morning Brief
📅 Tuesday, June 23, 2026

📊 10 New AI Stories
━━━━━━━━━━━━━━━━━━━

1️⃣ Retrieval Is Filtering, Not Search...
🔥 9/10 | AI Architecture | Positive
📝 This article proposes a mental model shift...
🏷️ #RAG #EnterpriseAI #DocumentIntelligence
🔗 Read More

2️⃣ The Era of No-Code AI...
...

━━━━━━━━━━━━━━━━━━━
🤖 Powered by ARIA
👨‍💻 Developed by Naveed Bhat
```

---

### 10. Telegram — Send a Text Message
- **Type:** Telegram node
- **Bot:** ARIA085_bot
- **Chat ID:** Your personal Telegram chat
- **Parse Mode:** Markdown (Legacy)
- **Delivers:** The formatted morning brief directly to your Telegram

---

## 📊 Google Sheets Database

**Sheet Name:** ARIA — AI News Database  
**Link:** [Open Sheet](https://docs.google.com/spreadsheets/d/1W0vdYCHY8e_8fjNHVTDxW_1PUxi7e8jORFBfTqn4rrg)

The sheet serves as ARIA's long-term memory — every article analyzed is stored with full metadata for future reference, newsletter creation, and trend analysis.

---

## 🤖 Telegram Bot

**Bot Name:** ARIA085_bot  
**Purpose:** Delivers the daily AI news brief  
**Chat:** Direct message to Naveed Bhat

**Sample output:**
- Top 5 AI stories of the day
- Relevance score for each (out of 10)
- Category, sentiment, trend tags
- Direct links to articles
- AI-generated 3-sentence summaries

---

## ⚙️ Configuration

### API Keys Required

| Service | Where to Get | Where to Put |
|---------|-------------|--------------|
| Google Gemini API | [aistudio.google.com](https://aistudio.google.com/app/apikey) | HTTP Request node URL `?key=...` |
| Telegram Bot Token | [@BotFather](https://t.me/BotFather) on Telegram | Telegram node credential |
| Google Sheets | n8n Google credential | Google Sheets node |

### Schedule Configuration
- Open Schedule Trigger node
- Set your preferred run times (default: 6 AM + 6 PM IST)
- Activate the workflow with the toggle (top right in n8n)

---

## 🚦 How to Run

### Manual Run
1. Open the ARIA workflow in n8n
2. Click **"Execute Workflow"** (orange button)
3. Wait ~2 minutes for completion
4. Check Google Sheet + Telegram

### Automatic Run
1. Ensure workflow is **Active** (toggle in top right)
2. ARIA runs automatically at scheduled times
3. No action required

---

## 📈 Performance

| Metric | Value |
|--------|-------|
| RSS feeds monitored | 14 |
| Articles collected per run | ~1,900 |
| Articles after AI filter | ~10 |
| Articles after quality gate | 7-10 |
| Primary AI | Gemini 2.0 Flash (**1** batch call) |
| Fallback AI | Groq Llama 3.1 8B (auto-activates) |
| Total run time | ~2 minutes |
| API quota used (daily) | ~2 requests/day |
| **Uptime** | **~100%** (dual AI redundancy) |

---

## 🔄 Data Flow Summary

```
14 RSS Feeds → ~1,900 articles
      ↓ AI keyword filter
    ~50 articles
      ↓ Deduplication
    ~30 unique articles
      ↓ Batch limit
    10 articles → Gemini API (1 call)
      ↓ AI analysis
    10 analyzed articles
      ↓ Quality filter (score ≥ 7)
    7-10 high-quality articles
      ↓
    Google Sheets (saved) + Telegram (delivered)
```

---

## 🛠️ Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| RSS node fails | Feed URL changed / 404 | Replace URL with working RSS |
| Gemini rate limit | Too many test runs | Wait 1 min; enabled auto-retry |
| JSON body invalid | Special chars in content | Code node pre-escapes with `JSON.stringify()` |
| Sheets not saving | Empty data from filter | Check Filter node score threshold |
| Telegram not sending | Wrong Chat ID or token | Verify credential in Telegram node |

---

## 🚀 Future Improvements

- [x] **Groq fallback** — auto-switches to Groq Llama 3.1 if Gemini fails ✅
- [ ] **Re-add deduplication** — check Google Sheet before saving (avoid same articles across runs)
- [ ] **Weekly digest** — summarize the week's top 10 articles every Sunday
- [ ] **Topic tracking** — track specific topics (e.g., "GPT-5", "Llama 4") over time
- [ ] **Source diversity scoring** — ensure variety across different publishers
- [ ] **Search interface** — query the database by keyword or date

---

## 📁 Project Structure

```
N8N/
├── README.md                          ← This file (full documentation)
├── ARIA — AI News Database.html       ← Google Sheets export reference
└── (n8n workflow runs in cloud)       ← naveedbhat.app.n8n.cloud
```

---

## 👨‍💻 About

**Project:** ARIA — AI Research Intelligence Assistant  
**Developer:** Naveed Bhat  
**Platform:** n8n Cloud  
**Built:** June 2026  
**Status:** ✅ Active & Running

> *"ARIA doesn't just collect news — it understands it."*
