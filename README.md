# Tess

A personal intelligence layer built on top of multiple language models — a router, a memory that learns from outcomes, and a critic that checks the models against each other before anything is called final.

## What Tess is

Tess is not a single trained model. She's an orchestration brain: a small codebase that sits on top of several existing LLM APIs (Groq, Nemotron, DeepSeek, Claude, or any others you plug in), and coordinates them into one consistent, improving system.

She has:
- **One voice** — a persona layer so every reply sounds like Tess, regardless of which model actually generated it underneath.
- **A router** — decides which model(s) a given question needs, instead of always calling everything.
- **A memory** — every interaction, including its eventual outcome, is stored and recalled the next time something similar comes up.
- **A synthesis + critic step** — when more than one model answers, Tess reconciles them and actively looks for reasons the answer could be wrong before treating it as final.

## What Tess is not

Tess is not a trading bot. She doesn't place trades, hold a broker connection, or manage capital. A separate project (`t_bot`) can optionally consume her output as one input among others — but that's a downstream integration, not what this repository is for. Keeping this boundary is deliberate: it keeps Tess debuggable as a general reasoning system, and keeps anything execution-related in its own codebase with its own risk controls.

## Why this architecture, and not one big trained model

No single person training a model from scratch on a laptop can compete with frontier labs on raw scale. That's not the goal here. The goal is a system that:

- Uses existing frontier models as raw material rather than trying to out-scale them
- Gets better over time through **retrieval**, not retraining — memory of past interactions and their outcomes, recalled at the right moment, rather than expensive fine-tuning runs
- Stays narrow and specific enough to be genuinely useful, rather than trying to be good at everything
- Runs entirely on API calls, so it's buildable and iterable on modest hardware (this was designed against an M1 MacBook Air)

## Architecture

```
                    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
                    │   Groq   │ │ Nemotron │ │ DeepSeek │ │  Claude  │
                    │  speed   │ │reasoning │ │math/code │ │ judgment │
                    └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘
                         └────────────┼─────────────┼────────────┘
                                      ▼
                    ┌─────────────────────────────────────────┐
                    │                 TESS CORE                │
                    │  ┌─────────┐  ┌─────────┐  ┌───────────┐ │
                    │  │ Persona │  │ Router  │  │  Memory   │ │
                    │  │  voice  │  │  model   │  │ context + │ │
                    │  │         │  │  select  │  │ reflection│ │
                    │  └─────────┘  └─────────┘  └───────────┘ │
                    └────────────────────┬──────────────────────┘
                                         ▼
                    ┌─────────────────────────────────────────┐
                    │           Synthesis + critic              │
                    │   Reconciles model answers, checks        │
                    │   consistency against memory               │
                    └────────────────────┬──────────────────────┘
                                         ▼
                    ┌─────────────────────────────────────────┐
                    │              Tess's answer                │
                    └───────┬─────────────────────┬─────────────┘
                            ▼                     ▼
                     Reply to user       Optional: forwarded to
                                          an external consumer
                                          (e.g. t_bot), as one
                                          structured signal among
                                          many inputs it uses
                            │
                            ▼ (once outcome is known)
                    ┌─────────────────────────────────────────┐
                    │        Written back into Memory           │
                    │     as an outcome + reflection record      │
                    └─────────────────────────────────────────┘
```

## Core concepts

**Persona** (`core/persona.py`)
A fixed system prompt defining Tess's voice, tone, and values. Every response is filtered through this regardless of which backend model produced the underlying reasoning, so she feels like one consistent entity.

**Router** (`core/router.py`)
Classifies an incoming query and decides which model(s) to call. Starts rule-based (cheap, fast, predictable) rather than using another LLM call for routing — upgrade later if needed.

**Council** (`core/council/`)
One thin client file per model provider. Each wraps that provider's API in a common interface so the rest of the codebase doesn't care which provider it's talking to.

**Memory** (`core/memory.py`)
A vector store (Chroma) holding past interactions. Two operations:
- *Retrieve*: before a call, pull the most similar past cases (including their outcome and reflection) into context.
- *Write*: after an outcome is known, store a structured record — what was asked, what was said, what happened, and a short reflection on why.

**Synthesis + critic** (`core/synthesis.py`)
When multiple models respond, this step reconciles them, flags disagreement instead of hiding it, and — for anything consequential — runs a dedicated pass that actively tries to find the flaw in the leading answer before it's accepted.

## Project structure

```
tess/
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── core/
│   ├── persona.py          # Tess's system prompt / voice
│   ├── router.py            # task classification, model selection
│   ├── memory.py             # vector store: retrieve, write, reflect
│   ├── synthesis.py           # reconciles multi-model answers, critic pass
│   └── council/
│       ├── base.py             # shared interface all clients implement
│       ├── groq_client.py
│       ├── nemotron_client.py
│       ├── deepseek_client.py
│       └── claude_client.py
├── api/
│   └── main.py               # FastAPI app: POST /ask
└── data/
    └── tess_memory/          # Chroma persistent store (gitignored)
```

## Setup

```bash
git clone <your-repo-url> tess
cd tess
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # then fill in your API keys
```

## Environment variables

```
GROQ_API_KEY=
NVIDIA_API_KEY=          # Nemotron via NVIDIA NIM
DEEPSEEK_API_KEY=
ANTHROPIC_API_KEY=
```

## Running locally

```bash
uvicorn api.main:app --reload --port 8000
```

Then:

```bash
curl -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"query": "What is Tess?"}'
```

## Roadmap

- **v1 (this repo)** — persona, router, single/multi-model council calls, memory store + retrieve, basic synthesis. Prove the full loop works end to end.
- **v2** — critic pass with a bounded retry loop, richer routing (confidence-based fan-out), structured output schemas per task type.
- **v3** — evaluate whether any narrow fine-tuning is worth it once enough real interaction history exists; keep retrieval as the default learning mechanism.

## License

MIT — do whatever you want with it.
