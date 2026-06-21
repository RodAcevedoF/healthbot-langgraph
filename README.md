# HealthBot

An AI-powered patient education prototype built as a [LangGraph](https://langchain-ai.github.io/langgraph/)
workflow. The patient picks a health topic; the bot searches reputable medical sources
with Tavily, summarizes them in plain language, quizzes the patient, grades the answers
with citations, suggests related topics, and loops or exits — resetting state between
topics for privacy.

The whole workflow lives in [`health_bot.ipynb`](health_bot.ipynb).

## Architecture

The graph is assembled from small, reusable pieces using four design patterns:

- **Reusable agent wrapper** - `LLMAgent` wraps the chat model (`system + history + user → text`).
- **Strategy** - each LLM step (summarize / quiz / grade / related) is a `PromptStrategy`;
  prompts read `difficulty` from state to scale depth.
- **Template method** - `Node` defines the call contract; `LLMNode` runs any strategy.
- **Builder** - `GraphBuilder` fluently registers nodes/edges and compiles the app.

```
ask_topic → ask_difficulty → ask_quiz_count → search → summarize → present_summary
  → make_quiz → present_quiz → evaluate → present_evaluation
        ├─ more questions → make_quiz   (loop until num_questions reached)
        └─ done → suggest_related → decision
                                      ├─ pick related / new topic → reset → ask_topic
                                      └─ exit → END
```

## Setup

This is a [uv](https://docs.astral.sh/uv/) project (Python 3.13).

```bash
uv sync                 # create the venv and install dependencies
cp .env.example .env    # then fill in your API keys
```

`.env` (git-ignored) must contain:

```
OPENAI_API_KEY="sk-..."
TAVILY_API_KEY="tvly-..."   # https://app.tavily.com/home (first 1000 requests free)
```

## Run

```bash
uv run jupyter lab health_bot.ipynb
```

Run the cells top to bottom: the graph image renders after the build cell, and the
interactive session starts at the final cell (it uses `input()`/`print()`).

## Layout

- `health_bot.ipynb` — the HealthBot workflow.
- `pyproject.toml` / `uv.lock` — dependencies.
- `.env.example` — template for your API keys.
- `specs/` — local project specs and rubric (git-ignored).
