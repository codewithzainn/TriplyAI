# TriplyAI — Multi-Agent AI Travel Planner

<p align="center">
  <img src="https://img.shields.io/badge/status-active-6366f1" alt="Status">
  <img src="https://img.shields.io/badge/python-3.11-a78bfa" alt="Python 3.11">
  <img src="https://img.shields.io/badge/framework-FastAPI-818cf8" alt="FastAPI">
  <img src="https://img.shields.io/badge/AI-LangGraph-7c3aed" alt="LangGraph">
  <img src="https://img.shields.io/badge/license-GPL--3.0-blue" alt="GPL-3.0">
</p>

> Describe your trip in plain English. TriplyAI coordinates a team of AI agents to research flights, hotels, weather, and build a day-by-day itinerary — all in one response.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Running the Application](#running-the-application)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [Attribution](#attribution)

---

## Overview

TriplyAI is a multi-agent AI travel planner built with **LangGraph** and **FastAPI**. It uses a supervisor–specialist agent pattern where each agent handles a specific part of trip planning (flights, hotels, weather, itinerary). Results are assembled into a single structured response with a day-by-day schedule and cost estimate.

Plans are saved to a PostgreSQL database so you can return to a previous trip and ask follow-up questions without losing context.

---

## Features

- **Multi-agent orchestration** via LangGraph — separate specialist agents for flights, hotels, weather, and itinerary
- **Conversational input** — describe trips in natural language, no forms required
- **Persistent trips** — conversations checkpointed to PostgreSQL; resumable at any time
- **Live data** — AviationStack (flights), Tavily (hotel/web search), OpenWeather (forecasts)
- **Trip Builder** — optional form-based prompt composer (origin, destination, dates, budget, interests)
- **Modern UI** — dark/light theme, glassmorphism design, fully responsive
- **Export** — copy, download as Markdown, or print any plan
- **Flexible API keys** — configure via `.env` or per-session through the settings panel

---

## Tech Stack

| Layer         | Technology                                           |
| ------------- | ---------------------------------------------------- |
| Backend       | Python 3.11, FastAPI, Uvicorn                        |
| AI / Agents   | LangGraph, LangChain, Groq (LLM inference)           |
| Agent Tools   | MCP (Model Context Protocol) servers                 |
| Data Sources  | AviationStack, Tavily, OpenWeather                   |
| Persistence   | PostgreSQL + LangGraph checkpoint postgres           |
| Caching       | Redis                                                |
| Frontend      | Vanilla HTML / CSS / JS, Jinja2 templates            |
| Deployment    | Docker, Vercel-compatible                            |

---

## Architecture

```
User Input
     │
     ▼
┌──────────────────────┐
│   Supervisor Agent   │  ← orchestrates the workflow
└──────────┬───────────┘
           │
     ┌─────┴──────────────────────────┐
     ▼              ▼                 ▼
Flight Agent    Hotel Agent    Weather Agent
(AviationStack) (Tavily)       (OpenWeather)
     │              │                 │
     └──────────────┴─────────────────┘
                    │
                    ▼
         Itinerary & Plan Agent
         (synthesises all results)
                    │
                    ▼
           Structured Plan → User
```

Each agent is a LangGraph node. The supervisor decides which agents to call and in what order based on the user's request. All conversation state is checkpointed to PostgreSQL so sessions are resumable.

---

## Getting Started

### Prerequisites

- Python 3.11
- [uv](https://docs.astral.sh/uv/) — Python package manager
- A PostgreSQL database (a free [Render](https://render.com/) instance works)
- API keys for the services listed below (all have free tiers)

### Clone and Install

```bash
git clone https://github.com/codewithzainn/TriplyAI.git
cd TriplyAI
uv sync
```

---

## Environment Variables

Create a `.env` file in the project root:

```dotenv
# Required
GROQ_API_KEY=your_groq_api_key
DATABASE_URL=postgresql://user:password@host:5432/dbname

# Service keys
TAVILY_API_KEY=your_tavily_key
AVIATIONSTACK_API_KEY=your_aviationstack_key
OPENWEATHER_API_KEY=your_openweather_key

# Optional
GROQ_MODEL=llama-3.3-70b-versatile
```

| Variable                | Required | Purpose                              |
| ----------------------- | :------: | ------------------------------------ |
| `GROQ_API_KEY`          | ✅        | Powers all AI agent reasoning        |
| `DATABASE_URL`          | ✅        | Persists trip conversations          |
| `TAVILY_API_KEY`        | ✅        | Hotel and destination search         |
| `AVIATIONSTACK_API_KEY` | ✅        | Airport and airline data             |
| `OPENWEATHER_API_KEY`   | ✅        | Live weather and forecasts           |
| `GROQ_MODEL`            | ❌        | Override the default LLM model       |

> ⚠️ The `.env` file is listed in `.gitignore` — never commit it.

Where to get each key:
- **Groq** → [console.groq.com](https://console.groq.com/)
- **Tavily** → [tavily.com](https://tavily.com/)
- **AviationStack** → [aviationstack.com](https://aviationstack.com/)
- **OpenWeather** → [openweathermap.org/api](https://openweathermap.org/api)

---

## Running the Application

**With uv (local development):**

```bash
uv run python app.py
```

**With Docker:**

```bash
docker-compose up --build
```

Open `http://127.0.0.1:8000` in your browser. A green *API connected* badge in the header confirms everything is working.

---

## Usage

**Planning a trip:**

Type your request in the input box and press Enter:

```
Plan a 7 day trip to Dubai from Karachi in December, mid-range budget
```

```
10 days in Europe from Lahore in April, covering London, Paris and Amsterdam
```

A plan takes 30–90 seconds to build. A progress bar shows the active stage.

**Trip Builder:**

Click the sliders icon next to the input box to open a structured form. Fill in origin, destination, dates, duration, budget and interests — then click **Write my prompt** to compose the request automatically.

**Saving and revisiting:**

- Every plan is saved automatically and persists across page reloads
- Reopen any past trip from the conversation history and continue asking questions
- Use the toolbar icons on any result to copy, download as Markdown, or print

---

## Project Structure

```
TriplyAI/
├── app.py                        # FastAPI entry point
├── frontend/
│   ├── templates/
│   │   └── index.html            # Main UI template
│   └── static/
│       ├── css/styles.css        # UI styles (Purple/Indigo theme)
│       └── js/
│           ├── app.js            # Frontend logic
│           └── markdown.js       # Markdown renderer
├── src/
│   ├── agents/                   # Specialist AI agents
│   │   ├── flight_agent.py
│   │   ├── hotel_agent.py
│   │   ├── weather_agent.py
│   │   ├── itinerary_agent.py
│   │   ├── final_agent.py
│   │   └── prompts.py
│   ├── api/                      # Session management, validation
│   ├── clients/                  # External API clients
│   ├── config/                   # Settings, credentials
│   ├── graph/                    # LangGraph workflow definition
│   ├── mcp_servers/              # MCP tool servers
│   └── utils/                    # Shared utilities
├── scripts/
├── .env.example
├── pyproject.toml
├── Dockerfile
└── docker-compose.yml
```

---

## Troubleshooting

**Planning fails after the page loads**

Check the status badge in the header. If it shows *API unreachable*, the server has stopped — restart it with `uv run python app.py` and check the terminal for errors.

**"Model does not exist" error**

Groq periodically retires models. List available models for your key:

```bash
curl -s https://api.groq.com/openai/v1/models \
  -H "Authorization: Bearer $GROQ_API_KEY"
```

Pick a model from the list and set it as `GROQ_MODEL` in your `.env`.

**"DATABASE_URL is missing" error**

The app requires a PostgreSQL connection string. Create a free database on [Render](https://render.com/) and add the URL to your `.env`.

**Plans take a long time**

30–90 seconds is normal. Multiple agents run in sequence, each making live API calls. The progress bar indicates which stage is active.

---

## Attribution

TriplyAI is built on top of the following open-source tools and services:

- [LangGraph](https://github.com/langchain-ai/langgraph) & [LangChain](https://github.com/langchain-ai/langchain) — agent orchestration
- [FastAPI](https://fastapi.tiangolo.com/) — web framework
- [Groq](https://groq.com/) — LLM inference
- [Tavily](https://tavily.com/) — web search API
- [AviationStack](https://aviationstack.com/) — flight data
- [OpenWeather](https://openweathermap.org/) — weather data

This project builds upon an open-source multi-agent AI travel planning project. The core LangGraph agent architecture, graph workflow pattern, and backend structure are derived from that work. The original project is licensed under the **GNU General Public License v3.0**, and this project retains that license accordingly.

See [LICENSE](LICENSE) for full license text.

---

*Built by **M. Zain Ul Abideen** — [github.com/codewithzainn](https://github.com/codewithzainn)*
