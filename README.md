# Agentic LangGraph

A Python learning project for building LangGraph agents and connecting them to
tools exposed by Model Context Protocol (MCP) servers. The current example
combines a local math server, a weather server, and an OpenAI chat model.

## What this project contains

- **LangGraph agent with MCP tools** (`src/agentic_langgraph/mcp/client.py`):
  discovers tools from the configured MCP servers and asks the agent to answer
  a math question and a weather question.
- **Math MCP server** (`src/agentic_langgraph/mcp/mathServer.py`): exposes
  `add` and `multiply` tools over standard input/output (stdio).
- **Weather MCP server** (`src/agentic_langgraph/mcp/weather.py`): exposes
  `getLatestWeather`, which looks up a place with Open-Meteo's geocoding API
  and fetches current conditions from its forecast API. It uses the
  streamable HTTP MCP transport.
- **Learning notebooks**
  (`src/agentic_langgraph/BasicChatbot/`): introductory LangGraph chatbot
  examples, including tools, ReAct-style agents, persistence, streaming, and
  human-in-the-loop workflows.
- **Python package entry point** (`src/agentic_langgraph/__init__.py`):
  currently prints a greeting; it does not start the MCP agent.

## Requirements

- Python 3.11 or newer.
- An OpenAI API key for the agent client.
- Internet access for the OpenAI and Open-Meteo API requests.
- The project dependencies installed in a virtual environment.

Dependencies and package metadata are defined in `pyproject.toml`; the project
also includes a `uv.lock` lockfile and a `requirements.txt` file.

## Setup on Windows

From the repository root in PowerShell:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e .
```

Alternatively, if you have [uv](https://docs.astral.sh/uv/) installed:

```powershell
uv sync
```

Create a `.env` file in the repository root and add your key:

```dotenv
OPEN_API_KEY=your-openai-api-key
```

The client loads `OPEN_API_KEY` from `.env` and assigns it to the
`OPENAI_API_KEY` variable used by `ChatOpenAI`. Keep `.env` private; it is
ignored by Git.

## Run the MCP example

The example uses two servers. Start the weather server first in its own
PowerShell terminal:

```powershell
Set-Location "src\agentic_langgraph\mcp"
python weather.py
```

It serves the weather MCP endpoint at `http://127.0.0.1:8000/mcp`. Keep this
terminal open while using the client.

In a second terminal, start the agent client from the same directory:

```powershell
Set-Location "src\agentic_langgraph\mcp"
python client.py
```

Starting the client from this directory matters because it launches the math
server using the relative path `mathServer.py`. The client connects to that
stdio server and the weather server, retrieves their tools, then submits
example math and weather questions to a `gpt-4o-mini` agent.

The weather server makes outbound requests to:

- `https://geocoding-api.open-meteo.com/v1/search`
- `https://api.open-meteo.com/v1/forecast`

If a request fails with a DNS or connection error, check that the machine can
resolve and reach those hosts and that the weather server is running locally.

## Run the package entry point

After installation, the project declares this console command:

```powershell
agentic-langgraph
```

At present, it only prints `Hello from agentic-langgraph!`. Use the MCP
instructions above to run the agent example.

## Project layout

```text
.
├── pyproject.toml
├── requirements.txt
├── uv.lock
└── src/
    └── agentic_langgraph/
        ├── __init__.py
        ├── BasicChatbot/
        │   ├── basicChatBot.ipynb
        │   └── humanInTheLoop.ipynb
        └── mcp/
            ├── client.py
            ├── mathServer.py
            └── weather.py
```

## Main dependencies

- **LangGraph** and **LangChain** — agent orchestration and chat-model
  integration.
- **langchain-mcp-adapters** and **mcp** — MCP client and server support.
- **langchain-openai** — OpenAI chat model integration.
- **python-dotenv** — loading local environment variables from `.env`.
- **httpx** — asynchronous HTTP requests to Open-Meteo.
- **ipykernel** — running the learning notebooks in Jupyter-compatible tools.

The `langchain-tavily` and `langsmith` packages are also included in the project
dependencies for further experiments.

## Notes

- The agent client currently contains fixed example questions and uses
  `gpt-4o-mini`.
- The weather API is public and does not require an API key.
- The weather server returns an explanatory message when a location cannot be
  resolved. Network and HTTP errors are not hidden.
- There are currently no documented automated test commands or test suite.