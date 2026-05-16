# SysAutopsy — Autonomous Multi-Agent Incident Investigation

SysAutopsy is an AI-powered post-mortem platform that autonomously investigates production incidents using a pipeline of specialist agents. It collects signals, generates hypotheses, tests evidence, debates conclusions, and produces a structured post-mortem report — all in real time.

---

## Demo

![SysAutopsy UI](https://i.imgur.com/placeholder.png)

> Select an incident → click **Investigate** → watch 5 AI agents reason through logs, metrics, GitHub deploys, Slack messages, and visual evidence in real time.

---

## Features

- **5-Agent Pipeline** — HypothesisAgent → EvidenceAgent → RootCauseAgent → CriticAgent → ReportAgent
- **Real-time SSE streaming** — every agent thought, tool call, and decision streams live to the UI
- **Adversarial Critic** — `qwen3-32b` independently challenges the root cause before the report is written
- **Visual Analysis** — drop a Grafana/Datadog screenshot for Gemini 2.5 Flash anomaly detection
- **Live URL Scraping** — paste any status page or post-mortem URL and investigate it instantly
- **AI Incident Generator** — generate realistic custom incidents for any service and type
- **PagerDuty Webhook** — simulate or receive real PD webhooks to auto-trigger investigations
- **Export to Markdown** — download a full structured post-mortem report
- **Investigation History** — all investigations persisted to SQLite with confidence scores

---

## Agent Architecture

```
Signals (logs, metrics, GitHub, Slack, PagerDuty, Vision)
        │
        ▼
 HypothesisAgent  ──►  EvidenceAgent  ──►  RootCauseAgent
 llama-3.3-70b         llama-3.1-8b         llama-3.3-70b
                                                  │
                                                  ▼
                                           CriticAgent  ──►  ReportAgent
                                           qwen3-32b          compound-beta
```

| Agent | Model | Role |
|---|---|---|
| HypothesisAgent | llama-3.3-70b-versatile | Generates & ranks hypotheses |
| EvidenceAgent | llama-3.1-8b-instant | Evaluates tool results against hypotheses |
| RootCauseAgent | llama-3.3-70b-versatile | Synthesizes confirmed evidence into root cause |
| CriticAgent | qwen-qwen3-32b | Adversarial review — finds logical gaps |
| ReportAgent | compound-beta | Writes the final post-mortem markdown |
| VisionAgent | gemini-2.5-flash | Analyzes dashboard screenshots for anomalies |

---

## Tech Stack

- **Backend** — Python 3.11+, FastAPI, Uvicorn
- **AI** — Groq (llama, qwen, compound-beta), Google Gemini 2.5 Flash
- **Frontend** — Vanilla HTML/CSS/JS (no framework), Server-Sent Events
- **Storage** — SQLite via aiosqlite
- **HTTP Client** — httpx (async)

---

## Project Structure

```
SysAutopsy/
├── main.py                 # FastAPI app, all routes
├── agent.py                # Multi-agent coordinator + specialist agents
├── live_tools.py           # /tools/* API endpoints (logs, metrics, github, slack)
├── mock_apis.py            # Incident loader + tool fallback implementations
├── vision_agent.py         # Gemini 2.5 Flash visual analysis
├── incident_generator.py   # AI-powered incident JSON generator
├── investigation_store.py  # SQLite persistence layer
├── prompts.py              # System prompts
├── incidents/
│   ├── incident_a.json     # P1 · API 500 Errors — OAuth · $38K
│   ├── incident_b.json     # P1 · Database Crash 2AM · $21K
│   ├── incident_c.json     # P1 · Silent 504s · $94K
│   ├── incident_d.json     # P1 · ML Memory Leak · $156K ★ Expert
│   └── incident_e.json     # P1 · Payment Cascade · $312K ★ Recommended
├── ui/
│   └── index.html          # Full single-page UI
├── requirements.txt
├── .env                    # API keys (never committed)
└── .gitignore
```

---

## Quickstart

### 1. Clone the repo

```bash
git clone https://github.com/praveenKUMAR-05/Anvil-Plugin_Minds.git
cd Anvil-Plugin_Minds
```

### 2. Create a virtual environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Set up environment variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
GOOGLE_API_KEY=your_google_api_key_here
```

- Get your Groq API key → https://console.groq.com
- Get your Google API key → https://aistudio.google.com/app/apikey

### 5. Run the server

```bash
uvicorn main:app --reload --port 8000
```

### 6. Open the UI

```
http://localhost:8000
```

---

## Usage

### Investigate a built-in incident
1. Select an incident from the dropdown
2. Click **Investigate**
3. Watch the 5-agent pipeline reason in real time across all panels

### Keyboard shortcuts
| Key | Action |
|---|---|
| `Space` | Load and investigate a random incident |
| `1` – `5` | Select incident A through E |

### Load a live URL
Paste any public status page, post-mortem blog post, or runbook URL into the **🌐 LIVE URL** bar and click **⬇ Load & Investigate**. The agent scrapes the page, extracts incident signals using Groq, and starts a full investigation.

### Generate a custom incident
Click **+ Generate**, enter a service name, incident type, and severity. Groq generates a realistic incident JSON and immediately starts investigating it.

### Visual analysis
Drop a PNG/JPG Grafana or Datadog screenshot onto the **Signal Feed** panel before clicking Investigate. Gemini 2.5 Flash will analyze it for anomalies and prepend findings to the signal feed.

### Simulate a PagerDuty webhook
Click **⚡ Webhook** to fire a simulated PagerDuty alert. The system auto-matches it to an incident and starts investigating without any user input.

### Export post-mortem
After investigation completes, click **⬇ Export Markdown** to download a full structured post-mortem `.md` file.

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Serve the UI |
| `GET` | `/incidents` | List all available incidents |
| `GET` | `/incidents/random` | Return a random incident ID |
| `GET` | `/investigate?incident_id=` | Stream investigation via SSE |
| `POST` | `/scrape-incident` | Scrape a URL and convert to incident |
| `POST` | `/generate-incident` | AI-generate a custom incident |
| `POST` | `/upload-screenshot` | Upload screenshot for Gemini analysis |
| `POST` | `/webhook/pagerduty` | Accept PagerDuty webhook payload |
| `GET` | `/history` | Last 20 investigation outcomes |
| `GET` | `/metrics` | Aggregate investigation statistics |
| `GET` | `/health` | Health check + uptime |
| `POST` | `/tools/logs` | Query incident logs |
| `POST` | `/tools/metrics` | Query incident metrics |
| `POST` | `/tools/github` | Query GitHub commits |
| `POST` | `/tools/slack` | Query Slack messages |

---

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `GROQ_API_KEY` | ✅ | — | Groq API key for all LLM agents |
| `GOOGLE_API_KEY` | ✅ | — | Google API key for Gemini VisionAgent |
| `MAX_CONCURRENT_INVESTIGATIONS` | ❌ | `10` | Max parallel investigations |
| `DB_PATH` | ❌ | `investigations.db` | SQLite database path |
| `TOOL_API_BASE_URL` | ❌ | `http://localhost:8000` | Base URL for tool API calls |
| `VULTR_DEPLOYMENT` | ❌ | `false` | Set `true` on Vultr VMs |

---

## Deployment

### Render (recommended — free tier)

1. Go to [render.com](https://render.com) → New → Web Service
2. Connect your GitHub repo
3. Set:
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn main:app --host 0.0.0.0 --port 8000`
4. Add environment variables: `GROQ_API_KEY`, `GOOGLE_API_KEY`
5. Click Deploy

### Railway

1. Go to [railway.app](https://railway.app) → New Project → Deploy from GitHub
2. Add environment variables
3. Railway auto-detects the start command

### Self-hosted / VPS

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

Set `VULTR_DEPLOYMENT=true` in `.env` to enable instance metadata in `/health`.

---

## Built-in Incidents

| ID | Title | Severity | Impact | Duration |
|---|---|---|---|---|
| `incident_a` | API 500 Errors — OAuth Algorithm Mismatch | P1 | $38,000 | 47 min |
| `incident_b` | Database Crash at 2AM — OOM Kill | P1 | $21,000 | 31 min |
| `incident_c` | Silent 504s — CDN Misconfiguration | P1 | $94,000 | 68 min |
| `incident_d` | ML Pipeline Memory Leak ★ Expert | P1 | $156,000 | 383 min |
| `incident_e` | Payment Cascade Failure ★ Recommended | P1 | $312,000 | 127 min |

---

## License

MIT
