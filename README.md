<h1 align="center">TriplyAI</h1>

<p align="center">
  <img src="https://img.shields.io/badge/status-active-6366f1.svg" alt="Status">
  <img src="https://img.shields.io/badge/built_by-M.%20Zain%20Ul%20Abideen-8b5cf6.svg" alt="Built by">
  <img src="https://img.shields.io/badge/python-3.11-a78bfa.svg" alt="Python">
  <img src="https://img.shields.io/badge/framework-FastAPI-818cf8.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/AI-LangGraph-7c3aed.svg" alt="LangGraph">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-GPL--3.0-blue.svg" alt="License"></a>
</p>

---

<p align="center">
  A <strong>multi-agent AI travel planner</strong> built with LangGraph and FastAPI.<br/>
  Describe your trip in plain English and get back a complete plan — flights, hotels,<br/>
  weather forecast and a day-by-day itinerary — researched and assembled in under a minute.
</p>

---

## 📝 Table of Contents

- [About](#about)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [Attribution](#attribution)

---

## 🧐 About <a name="about"></a>

**TriplyAI** solves the most painful part of trip planning: the research.

Planning a trip usually means juggling half a dozen browser tabs — one for flights, another for hotels, a weather check, and a notes app to piece it all together. TriplyAI collapses that entire process into a single conversation. You describe the trip you want in your own words:

> *"Plan a 7 day trip to Dubai from Karachi in December, mid-range budget"*

A team of specialised AI agents then goes to work in parallel. One researches flights, another finds hotels, another checks the weather. Their findings are assembled into a single, actionable plan complete with a day-by-day itinerary and cost estimate — in about a minute.

| | |
|---|---|
| ✈️ **Flights** | Likely airports, airlines on the route, typical duration and fare range |
| 🏨 **Hotels** | Accommodation options matched to your destination and budget |
| 🌤️ **Weather** | Current conditions, the forecast, and travel advice |
| 🗺️ **Itinerary** | A realistic day-by-day plan you can actually follow |
| 💰 **Budget** | An estimated breakdown of what the trip will cost |

Plans are saved automatically so you can reopen a trip and ask follow-up questions without starting over.

---

## ✨ Features <a name="features"></a>

- 🤖 **Multi-Agent Architecture** — Specialised agents for flights, hotels, weather, and itinerary planning run as a coordinated LangGraph graph
- 💬 **Conversational Interface** — Natural language input; no forms to fill
- 🔁 **Persistent Trip Memory** — Conversations saved to PostgreSQL via LangGraph checkpointer; ask follow-up questions on any saved trip
- ⚡ **Real-Time Data** — Live flight info via AviationStack, hotel search via Tavily, weather via OpenWeather
- 🎨 **Modern UI** — Glassmorphism dark/light interface, smooth animations, fully responsive
- 🛠️ **Trip Builder** — Optional structured form to compose prompts from fields (origin, destination, dates, budget, interests)
- 📥 **Export Plans** — Copy, download as Markdown, or print any plan
- 🔑 **Flexible Credentials** — API keys via `.env` for shared deployments or per-session via the settings panel

---

## 🛠️ Tech Stack <a name="tech-stack"></a>

| Layer | Technology |
|---|---|
| **Backend** | Python 3.11, FastAPI, Uvicorn |
| **AI / Agents** | LangGraph, LangChain, Groq (LLM inference) |
| **Agent Tools** | MCP (Model Context Protocol) servers |
| **Data Sources** | AviationStack (flights), Tavily (hotels/web), OpenWeather |
| **Persistence** | PostgreSQL + LangGraph checkpoint postgres |
| **Frontend** | Vanilla HTML/CSS/JS, Jinja2 templates |
| **Caching** | Redis |
| **Deployment** | Docker, Vercel-compatible |

---

## 🏗️ Architecture <a name="architecture"></a>

TriplyAI uses a **supervisor + specialist** multi-agent pattern built on LangGraph:

```
User Input
    │
    ▼
┌─────────────────────────────────┐
│        Supervisor Agent         │  ← orchestrates the workflow
└────────────┬────────────────────┘
             │ routes to specialists
    ┌────────┴──────────────────────────┐
    │                                   │
    ▼                                   ▼
Flight Agent         Hotel Agent    Weather Agent
(AviationStack)      (Tavily)       (OpenWeather)
    │                   │               │
    └───────────────────┴───────────────┘
                        │
                        ▼
              Itinerary & Plan Agent
              (synthesises all results)
                        │
                        ▼
              Final structured plan → User
```

Each agent is a LangGraph node. The supervisor decides which agents to invoke and in what order, based on the user's request. All state is checkpointed to PostgreSQL so conversations are resumable.

---

## 🏁 Getting Started <a name="getting-started"></a>

### Prerequisites

- **Python 3.11**
- **[uv](https://docs.astral.sh/uv/)** — fast Python package manager
- **A PostgreSQL database** — a free [Render](https://render.com/) instance works fine
- API keys from the services below (all have free tiers):
  - [Groq](https://console.groq.com/) — LLM inference
  - [Tavily](https://tavily.com/) — Hotel/web search
  - [AviationStack](https://aviationstack.com/) — Flight data
  - [OpenWeather](https://openweathermap.org/api) — Weather data

### Installing

Clone the repository and move into it:

```bash
git clone https://github.com/your-username/triplyai.git
cd triplyai
```

Install dependencies:

```bash
uv sync
```

Create a `.env` file in the project root:

```dotenv
# Required
GROQ_API_KEY=your_groq_api_key
DATABASE_URL=postgresql://user:password@host:5432/dbname

# Service keys
TAVILY_API_KEY=your_tavily_key
AVIATIONSTACK_API_KEY=your_aviationstack_key
OPENWEATHER_API_KEY=your_openweather_key

# Optional — switch LLM model without editing code
GROQ_MODEL=llama-3.3-70b-versatile
```

| Variable | Required | Purpose |
|---|:---:|---|
| `GROQ_API_KEY` | ✅ | Powers all AI agent reasoning |
| `DATABASE_URL` | ✅ | Persists trip conversations |
| `TAVILY_API_KEY` | ✅ | Hotel and destination search |
| `AVIATIONSTACK_API_KEY` | ✅ | Airport and airline data |
| `OPENWEATHER_API_KEY` | ✅ | Live weather and forecasts |
| `GROQ_MODEL` | ❌ | Override the default LLM model |

> ⚠️ **Never commit your `.env` file.** It is already listed in `.gitignore`.

Start the app:

```bash
uv run python app.py
```

Open **<http://127.0.0.1:8000>** in your browser. A green *API connected* dot in the header confirms everything is working.

### Running with Docker

```bash
docker-compose up --build
```

---

## 🎈 Usage <a name="usage"></a>

### Planning a trip

Type your request and press **Enter** (or click the send button). Anything natural works:

> Plan a 10 day Europe trip from Lahore in April, mid-range budget

> I want a relaxed 5 day beach holiday in Thailand for two people under $1500

Not sure what to type? Click any **suggestion card** on the home screen to get started instantly.

A plan takes **30–90 seconds** to build. You will see a live progress bar as the agents work.

### Using the Trip Builder

Click the **sliders icon** to the left of the message box to open the trip builder form. Fill in your origin, destination, dates, duration, number of travellers, budget, and interests — then click **Write my prompt**. Your request is composed automatically, ready to send or edit.

### Reading your plan

The plan appears as a single structured response containing:

- **Overview** — summary of the trip
- **Flights** — route, airline options, estimated fares
- **Hotels** — recommended properties with price ranges
- **Weather** — forecast for your travel dates with advice
- **Itinerary** — day-by-day schedule

### Saving and revisiting

- Every plan is automatically saved — reload the page and your history persists
- Ask follow-up questions on an open trip and the agents remember the context
- Use the icons at the top of any result to **copy**, **download as Markdown**, or **print**
- Click **New trip** to start a fresh conversation

---

## 📁 Project Structure <a name="project-structure"></a>

```
triplyai/
├── app.py                  # FastAPI entry point
├── frontend/
│   ├── templates/
│   │   └── index.html      # Main UI template
│   └── static/
│       ├── css/styles.css  # All UI styles (Purple/Indigo theme)
│       └── js/
│           ├── app.js      # Frontend logic
│           └── markdown.js # Markdown renderer
├── src/
│   ├── agents/             # Specialist AI agents (flight, hotel, weather, etc.)
│   ├── api/                # Session management, validation
│   ├── clients/            # External API clients (AviationStack, OpenWeather)
│   ├── config/             # Settings, credentials
│   ├── graph/              # LangGraph workflow definition
│   ├── mcp_servers/        # MCP tool servers
│   └── utils/              # Shared utilities
├── scripts/                # Helper scripts
├── .env.example            # Example environment file
├── pyproject.toml          # Python project config
├── Dockerfile              # Docker container config
└── docker-compose.yml      # Docker Compose setup
```

---

## 🤔 Troubleshooting <a name="troubleshooting"></a>

<details>
<summary><b>The page loads but planning fails</b></summary>

Check the status indicator in the header. If it says *API unreachable*, restart the server:

```bash
uv run python app.py
```

Otherwise check the terminal for the error traceback.
</details>

<details>
<summary><b>An error says the model does not exist</b></summary>

Groq periodically retires models. List available models for your key:

```bash
curl -s https://api.groq.com/openai/v1/models \
  -H "Authorization: Bearer $GROQ_API_KEY"
```

Pick one and set it as `GROQ_MODEL` in your `.env`.
</details>

<details>
<summary><b>DATABASE_URL is missing</b></summary>

The app needs a PostgreSQL database. Create a free one on [Render](https://render.com/) and add the connection string to `.env`.
</details>

<details>
<summary><b>Plans take a long time</b></summary>

30–90 seconds is normal. Several agents run in sequence, each making live API calls. The progress bar shows you which stage is active.
</details>

---

## 📜 Attribution <a name="attribution"></a>

TriplyAI is built on top of excellent open-source tools and services:

- [LangGraph](https://github.com/langchain-ai/langgraph) & [LangChain](https://github.com/langchain-ai/langchain) — agent orchestration framework
- [Groq](https://groq.com/) — fast LLM inference
- [Tavily](https://tavily.com/) — web/hotel search API
- [AviationStack](https://aviationstack.com/) — flight data API
- [OpenWeather](https://openweathermap.org/) — weather data API
- [FastAPI](https://fastapi.tiangolo.com/) — web framework
- This project is inspired by and builds upon open-source work in the multi-agent AI travel planning space. The core agent architecture, LangGraph workflow pattern, and original project structure were derived from an open-source project licensed under the **GNU General Public License v3.0**.

---

Licensed under the **GNU General Public License v3.0**. See [LICENSE](LICENSE).

**Built by M. Zain Ul Abideen** — as a portfolio project demonstrating multi-agent AI system design with LangGraph, FastAPI, and real-time data integration.
