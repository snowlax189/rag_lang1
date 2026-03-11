# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Jupyter notebook-based course repo for "AI Agents in LangGraph" (DeepLearning.AI). All code lives in four notebooks — there is no build system, package.json, or test suite.

## Running Notebooks

Open and run with Jupyter:
```bash
jupyter notebook
```

Or a specific notebook:
```bash
jupyter notebook Lesson_1_Student.ipynb
```

## Environment Setup

Install dependencies:
```bash
pip install -r requirements.txt
```

Copy `.env` and fill in your keys before running any notebook:
```
COHERE_API_KEY=        # required for all LLM calls (L1, L2, L4)
TAVILY_API_KEY=        # required for web search (L2, L3, L4)
OPENAI_API_KEY=        # kept for reference — not used
```

All notebooks call `load_dotenv()` at the top to load these automatically.

## Architecture

The LLM backend is **Cohere** (native SDK and LangChain integration):
- **Lesson 1** — Uses `cohere.ClientV2` directly. Implements a ReAct agent from scratch with manual action parsing via regex.
- **Lesson 2** — Rebuilds the same agent using LangGraph (`StateGraph`) + `langchain_cohere.ChatCohere` + `TavilySearchResults` for web search. Uses `model.bind_tools()` for function calling.
- **Lesson 3** — No LLM. Demonstrates agentic search directly via `TavilyClient` SDK, DuckDuckGo (`DDGS`), and `BeautifulSoup` scraping. No Cohere key needed.
- **Lesson 4** — Extends Lesson 2 with persistence (`SqliteSaver` / `AsyncSqliteSaver`) and async token streaming via `abot.graph.astream_events()`.

### Model mapping
| Use case | Model |
|----------|-------|
| Lightweight / fast | `command-r` |
| Complex reasoning | `command-a-03-2025` |

### Key LangGraph pattern (L2 & L4)
```
StateGraph: llm → (conditional) → action → llm → ...
```
The `exists_action` edge checks `tool_calls` on the last message; if present, routes to `action` node (tool execution), otherwise ends.
