# PackYourBags

An AI-powered travel planning app built with **LangGraph**, **MCP servers**, **Groq**, **Tavily**, **AviationStack**, **OpenWeather**, **PostgreSQL**, and **Streamlit**.

**Live app:** https://packyourbags.streamlit.app/  
**Repository:** https://github.com/masked-byte18/PackYourBags

The app takes a natural-language travel request such as:

```text
Plan a complete 7 day Japan trip including flights, hotels and sightseeing under 2 lakhs.
```

Then a set of specialized agents work together to collect flight guidance, hotel research, weather data, and a final itinerary.

---

## What We Built

PackYourBags is a multi-agent travel planner. Instead of asking one LLM to do everything, the project breaks travel planning into focused steps:

- A **Flight Agent** checks aviation data and creates flight guidance.
- A **Hotel Agent** uses Tavily search to find hotel and stay information.
- A **Weather Agent** calls a local MCP weather server backed by OpenWeather.
- An **Itinerary Agent** combines everything into a practical travel plan.
- **PostgreSQL memory** stores LangGraph checkpoints so sessions can persist.
- A **Streamlit UI** gives the user a simple web app experience.

In easy words: you type your trip idea, and the system behaves like a small travel team that researches flights, hotels, weather, and itinerary details before giving you a final plan.

---

## Visual Overview

```mermaid
flowchart TD
    User[User Travel Request] --> UI[Streamlit Frontend]
    UI --> Graph[LangGraph Workflow]

    Graph --> Flight[Flight Agent]
    Graph --> Hotel[Hotel Agent]
    Graph --> Weather[Weather Agent]
    Graph --> Itinerary[Itinerary Agent]

    Flight --> AviationMCP[AviationStack MCP Server]
    Hotel --> Tavily[Tavily MCP Search]
    Weather --> WeatherMCP[Local Weather MCP Server]
    WeatherMCP --> OpenWeather[OpenWeather API]

    AviationMCP --> AviationAPI[AviationStack API]
    Itinerary --> Groq[Groq LLM]
    Flight --> Groq

    Graph <--> Postgres[(PostgreSQL Checkpointer)]
    Itinerary --> Final[Final Travel Plan]
    Final --> UI
```

---

## Agent Workflow

```mermaid
sequenceDiagram
    participant U as User
    participant S as Streamlit
    participant G as LangGraph
    participant F as Flight Agent
    participant H as Hotel Agent
    participant W as Weather Agent
    participant I as Itinerary Agent

    U->>S: Enters travel request
    S->>G: Sends request state
    G->>F: Generate flight guidance
    F-->>G: Flight information
    G->>H: Search hotel options
    H-->>G: Hotel research
    G->>W: Fetch weather and forecast
    W-->>G: Weather data
    G->>I: Combine all context
    I-->>G: Final itinerary
    G-->>S: Stream updates and final result
    S-->>U: Display travel plan
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | Streamlit |
| Agent Orchestration | LangGraph |
| LLM | Groq LLaMA model |
| Search | Tavily MCP |
| Flight Data | AviationStack MCP |
| Weather Data | Local MCP server + OpenWeather |
| Memory / Checkpointing | PostgreSQL + `langgraph-checkpoint-postgres` |
| Runtime | Python |

---

## Project Structure

```text
PackYourBags/
├── frontend.py                         # Streamlit web app
├── main.py                             # LangGraph agent workflow
├── mcp_client.py                       # MCP client configuration and tool calls
├── custom_weather_mcp_server.py        # Local Weather MCP server
├── testing_aviationstack_mcp_server.py # Aviation MCP test script
├── tesing_weather_mcp_server.py        # Weather MCP test script
├── requirements.txt                    # Python dependencies
├── README.md                           # Project documentation
└── .env                                # Local secrets, not committed
```

The `aviationstack-mcp/` folder is downloaded locally during setup and is ignored by Git because it is an external dependency.

---

## Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key
OPENWEATHER_API_KEY=your_openweather_api_key
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/langgraph_memory
```

Do not commit `.env`. It contains private API keys.

---

## Setup

### 1. Clone The Repository

```bash
git clone https://github.com/masked-byte18/PackYourBags.git
cd PackYourBags
```

### 2. Create And Activate Virtual Environment

Git Bash:

```bash
python -m venv .venv
source .venv/Scripts/activate
```

PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\activate
```

### 3. Install Dependencies

```bash
python -m pip install -r requirements.txt
```

### 4. Install AviationStack MCP Server

```bash
git clone https://github.com/Pradumnasaraf/aviationstack-mcp.git aviationstack-mcp
cd aviationstack-mcp
uv sync
cd ..
```

`uv sync` creates the AviationStack MCP virtual environment used by `mcp_client.py`.

### 5. Setup PostgreSQL

Create a database for LangGraph memory:

```sql
CREATE DATABASE langgraph_memory;
```

Then update `DATABASE_URL` in `.env`.

Example:

```env
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/langgraph_memory
```

If PostgreSQL is unavailable, the app can still run without persistent memory, but long-term checkpointing will be disabled.

---

## Run The App

### Streamlit Web App

Git Bash:

```bash
cd "/c/Users/rajka/Desktop/Web_Dev/Agentic Projects/PackYourBags"
./.venv/Scripts/python.exe -m streamlit run frontend.py
```

PowerShell:

```powershell
cd "C:\Users\rajka\Desktop\Web_Dev\Agentic Projects\PackYourBags"
.\.venv\Scripts\python.exe -m streamlit run frontend.py
```

Open:

```text
http://localhost:8501
```

### Terminal Mode

```bash
./.venv/Scripts/python.exe main.py
```

---

## Important Runtime Notes

You only need to start the Streamlit app manually.

The MCP servers are started automatically when agents need them:

- `custom_weather_mcp_server.py` runs as the local weather MCP server.
- `aviationstack-mcp` runs as the local aviation MCP server.
- Tavily MCP is remote and connects through the Tavily API key.

PostgreSQL should be running in the background as a local service.

---

## Example Prompts

```text
Plan a 7 day Japan trip under 2 lakhs including flights, hotels, food, sightseeing, and weather.
```

```text
Plan a 5 day Paris trip for a couple with mid-range hotels and must-visit attractions.
```

```text
Create a Dubai weekend itinerary with flights, hotels, weather, and budget tips.
```

---

## Features

- Multi-agent workflow using LangGraph
- Streamlit web interface
- Live hotel research through Tavily
- Aviation guidance through AviationStack MCP
- Weather and forecast through OpenWeather MCP
- Groq LLM for travel reasoning and itinerary generation
- PostgreSQL checkpointing for persistent graph state
- Markdown travel plan generation and download
- Local-first setup with secrets stored in `.env`

---

## How The Code Works

### `frontend.py`

Builds the Streamlit interface. It collects the user query, calls the LangGraph app, streams agent updates, displays the final plan, and saves generated plans into `travel_plans/`.

### `main.py`

Defines the LangGraph workflow. It creates the travel state, registers the agents, connects the graph edges, configures PostgreSQL checkpointing, and exposes the compiled `app`.

### `mcp_client.py`

Configures MCP connections:

- Tavily remote MCP over HTTP
- AviationStack local MCP over stdio
- Weather local MCP over stdio

It also exposes helper functions used by the agents.

### `custom_weather_mcp_server.py`

Defines weather tools using MCP:

- `get_current_weather`
- `get_forecast`

These tools call OpenWeather and return structured weather data.

---

## Troubleshooting

### `ModuleNotFoundError: No module named 'psycopg'`

You are running the wrong Python or wrong Streamlit executable.

Use:

```bash
./.venv/Scripts/python.exe -m streamlit run frontend.py
```

Do not use plain `streamlit run frontend.py` if it points to Anaconda or global Python.

### OpenWeather Returns `401 Invalid API key`

Make sure:

- `OPENWEATHER_API_KEY` is from openweathermap.org.
- The key has activated. New keys can take a few minutes.
- There are no spaces around the key in `.env`.

### PostgreSQL Connection Fails

Check:

- PostgreSQL service is running.
- Port is correct, usually `5432`.
- Username, password, and database name match `DATABASE_URL`.

### AviationStack MCP Fails

Run:

```bash
cd aviationstack-mcp
uv sync
cd ..
```

Then restart Streamlit.

---

## Security Notes

- Never commit `.env`.
- Rotate API keys if they are accidentally shared.
- `.gitignore` excludes `.env`, `.venv`, generated plans, caches, and the external AviationStack repo.

---

## Repository

```text
https://github.com/masked-byte18/PackYourBags
```

---

## Summary

PackYourBags is a practical AI travel planning system. It combines a friendly Streamlit interface with a LangGraph multi-agent backend and MCP-powered external tools. The result is a working travel assistant that can research flights, hotels, weather, and generate complete trip itineraries from a simple prompt.
