# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the **LangChain Academy** course repository — an educational set of Jupyter notebooks teaching LangGraph concepts. It is **not** a library or application; the primary artifacts are notebooks and companion graph definitions for LangGraph Studio.

## Project structure

- `module-0/` — Python/LangChain basics
- `module-1/` — Core LangGraph: simple graphs, routers, chains, agents
- `module-2/` — State management: schemas, reducers, memory, message trimming
- `module-3/` — Human-in-the-loop: breakpoints, state editing, time travel
- `module-4/` — Advanced patterns: parallelization, sub-graphs, map-reduce, research assistant
- `module-5/` — Long-term memory: memory agents, memory stores, profile/collection schemas
- `module-6/` — Deployment: LangGraph Cloud, SDK, double-texting

Each module's `studio/` subdirectory contains:
- Python files exporting a `graph` variable (compiled `StateGraph`)
- `langgraph.json` — graph registry used by `langgraph dev`
- `.env.example` / `.env` — API keys for the graphs

## Setup and commands

```bash
# Create and activate virtual environment
python3 -m venv lc-academy-env
source lc-academy-env/bin/activate
pip install -r requirements.txt

# Run notebooks
jupyter notebook

# Run LangGraph Studio locally (from a module's studio/ directory)
cd module-X/studio
langgraph dev
# Studio UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
```

## Required environment variables

- `GOOGLE_API_KEY` — used by all modules (ChatGoogleGenerativeAI with gemini)
- `LANGSMITH_API_KEY`, `LANGSMITH_TRACING_V2="true"`, `LANGSMITH_PROJECT="langchain-academy"` — LangSmith tracing
- `TAVILY_API_KEY` — web search, used in module-4 and module-6

## Key patterns in the codebase

- **Graph construction**: All graphs follow `StateGraph(SomeTypedDict) → add_node → add_edge → compile()` with the compiled graph exported as `graph`.
- **State types**: `MessagesState` (from langgraph) for chat-based graphs; custom `TypedDict` subclasses for domain-specific state. `Annotated[list, operator.add]` is used as a reducer for accumulating values.
- **Structured output**: `llm.with_structured_output(PydanticModel)` is used extensively for extracting typed data from LLM responses.
- **Sub-graphs**: Module 4's research assistant compiles an inner `interview_builder` graph and nests it as a node in the outer `ResearchGraphState` graph.
- **Send() API**: Used for dynamic fan-out (e.g., spawning parallel interviews per analyst in module-4).
- **Trustcall**: Module 5-6 use the `trustcall` library's `create_extractor` for memory updates with patch-based document editing.
- **Configuration**: Module 5-6 use a `Configuration` dataclass with `from_runnable_config()` for per-user configurable fields.
- **Human-in-the-loop**: `compile(interrupt_before=[...])` is used to pause graphs for human feedback (module-3, module-4).
