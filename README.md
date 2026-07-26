# Tess

*A personal intelligence layer — one identity, a real memory, and a voice of her own.*

## The Dream

Most AI is stateless — every conversation starts from zero, indifferent to who you are or what came before. Tess is the opposite premise: a single, persistent intelligence that belongs to one person, remembers everything that matters, learns from what worked and what didn't, and grows more useful with every interaction instead of forgetting it the moment the session ends.

She's not built by training a model from scratch — that's not where the leverage is for a single builder. She's built by orchestrating the best available models like instruments in an ensemble, and layering on top of them the three things no raw API call has: a permanent identity, a memory that actually shapes future decisions, and — soon — a voice.

The long-term goal isn't a chatbot that answers when spoken to. It's a system that understands context deeply enough to anticipate what's needed, and eventually becomes the intelligence layer behind real, specialized work — starting with trading research and decision support for the `t_bot` project.

## What Tess Does

- **Holds a permanent identity.** One consistent voice and judgment, regardless of which model is generating the words underneath in any given moment.
- **Remembers, and learns from outcomes.** Every interaction — and, once known, its result — is stored and recalled the next time something similar comes up. Reflection compounds over time without ever retraining a model.
- **Routes intelligently across models.** Different tasks go to whichever model is best suited to them, drawing from several frontier-grade open models rather than being limited to one.
- **Reaches the live internet.** Not confined to frozen training data — she can pull current information into her reasoning before answering.

## Architecture

Four systems, each with a single clear responsibility, designed to extend cleanly as new capability — voice, proactive reasoning, specialized domains — gets added:

```
                          ┌────────────────────────┐
                          │          app.py           │
                          │   The interface — chat      │
                          │   today, voice tomorrow       │
                          └────────────┬───────────────┘
                                       ▼
                          ┌────────────────────────┐
                          │         brain.py           │
                          │  Identity, judgment, and     │
                          │  the reasoning loop that       │
                          │  ties every system together      │
                          └──────┬─────────────────┬─────────┘
                                 ▼                 ▼
                     ┌─────────────────┐  ┌─────────────────────┐
                     │    memory.py       │  │      models.py         │
                     │  Persistent record   │  │  Model routing and       │
                     │  of every interaction │  │  live web access          │
                     │  and its outcome        │  │                            │
                     └─────────────────┘  └─────────────────────┘
```

**`models.py`** — the connection layer. Presently routes across several free, frontier-grade open models hosted on Groq (a strong general model, a top-tier reasoning model, and a dedicated math/reasoning specialist), plus live web search. New capability — a paid model, a specialized tool, a voice API — plugs in here without touching anything else.

**`memory.py`** — the record of everything Tess has done and learned. Built on vector retrieval, so the most relevant past experience surfaces automatically, not just the most recent.

**`brain.py`** — Tess herself. Her fixed identity, and the reasoning loop that decides what she needs (memory? live search? which model?) before producing an answer.

**`app.py`** — how you actually reach her. A real chat interface today; the entry point for voice in the next phase.

## Setup

```bash
git clone <your-repo-url> tess
cd tess
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # add your free API keys
streamlit run app.py
```

## Engineering Principles

The standard this is held to, non-negotiable regardless of what gets added later:

- Every claim about memory is grounded in what was actually retrieved — never invented for the sake of sounding coherent.
- A system status is confirmed by testing it directly, never by asking Tess to self-report.
- One failing component degrades gracefully rather than taking the whole system down.
- Simple and understood beats sophisticated and opaque, at every stage of this build.

## Roadmap

**Phase 1 — Foundation (current)**
Identity, memory, multi-model routing, and live search, working end to end on modest hardware.

**Phase 2 — Voice**
A natural, human-paced voice layer, so Tess is spoken to and answers back, not just typed at.

**Phase 3 — Proactive intelligence**
Moving from reactive Q&A toward genuinely anticipating what's needed from context, not only from direct instruction.

**Phase 4 — Specialized domains**
Extending Tess into focused applications, beginning with `t_bot` — Tess as the research and reasoning layer, working alongside dedicated, separately engineered execution and risk systems built specifically for trading.

## License

MIT.
