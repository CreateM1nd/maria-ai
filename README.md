# Maria — Personal AI Agent System

---

## Background

I spent years in high-precision environments — autonomous vehicle perception testing at Uber ATG, service operations, mechanical diagnostics. The throughline in all of it was the same discipline: build systems that stay reliable when inputs are unpredictable. Find the edge cases before they find you.

When I transitioned into AI infrastructure, I brought that lens with me. Maria is what came out of it — a locally-hosted, multi-agent AI system I designed, built, and run in production every day. Not a tutorial project. A real environment where I work on hard problems.

---

## What I Learned

The hard problems in AI agent systems aren't the models — it's the **infrastructure around them**.

Getting an LLM to answer a question is easy. Getting an agent to:
- Stay coherent across 100-message sessions
- Recall relevant context from weeks ago without hallucinating
- Verify its own outputs before reporting them
- Operate autonomously without drifting from its original instructions
- Diagnose and recover from context drift and memory retrieval failures
- Stop after a successful action instead of spiraling into re-evaluation
- Know which tool to call without inventing names that sound plausible
- Distinguish between "I remember X" (file record exists) and "I think X" (inference)

...that is engineering. The same class of problems I worked on testing autonomous vehicle perception systems — where a model that works 99% of the time is not good enough, and the 1% edge cases are exactly what you are there to find.

The domain changed. The discipline did not.

---

## What It Does

Maria is a personal AI infrastructure layer — a system of cooperating agents with persistent memory, a searchable knowledge base, and autonomous scheduling. It runs entirely on local hardware with no cloud dependency for inference.

- **Two specialized agents** with distinct roles, models, and context budgets
- **Five-tier memory stack** combining vector recall, structured wiki injection, daily logs, and curated long-term storage
- **Autonomous heartbeat** — Maria checks in every 30 minutes during active hours, independent of user input
- **681 ingested sources** — 35 books across security, systems, AI/ML, and Python, plus architecture diagrams, identity files, and project docs
- **Behavioral protocols** enforced at the system level — not just prompts — to close gaps that soft instructions leave open

---

## Two-Agent Design

- **Maria** — primary agent running Gemma 4 12B. Handles conversation, task execution, memory management, and autonomous heartbeat checks. Context window: 220K tokens.
- **Rex** — reasoning specialist running DeepSeek-R1 14B. Delegated for deep analysis, structured thinking, and complex inference tasks. Context window: 32K tokens.

---

## Memory Architecture

Maria runs a five-tier memory stack:

| Tier | System | What it stores |
|---|---|---|
| 1 | In-context | Live session — dies on session end |
| 2 | LanceDB vector (memory-relay) | Auto-captured preferences, decisions, identity markers |
| 3 | Wiki sources (memory-core) | 681 ingested .md files — books, identity files, diagrams |
| 4 | Daily logs |  — raw session events |
| 5 | MEMORY.md | Curated long-term distillation — manually verified facts |

**Dual injection:** Both memory-core (wiki chunks) and memory-relay (LanceDB vector recall) fire their  hooks on every turn, injecting relevant context before the model sees the message.

**Auto-capture:** The  hook scans up to 3 user messages per turn and stores preferences, decisions, and identity markers to LanceDB automatically — with an injection guard that blocks prompt injection attempts before storage.

**Dreaming system:** Nightly consolidation pipeline (REM → Light → Deep) promotes high-confidence short-term memories to  and prunes stale entries. Dream diary written to .

**Memory integrity policy:** "I remember X" means a file record exists. "It is logged" means a tool call confirmed the entry. User claims are not verification. An honest "I do not know" beats a confident wrong answer.

---

## Behavioral Protocols

Maria operates under a set of hardcoded behavioral rules — not prompts, rules. Prompting says "please do X." Protocols fire regardless of context.

**Commit Protocol** — once a tool call returns success, the decision is closed. Confirm in one sentence and stop. No re-evaluation, no alternatives. The loop-break test: if the tool succeeded, you are done.

**Document Review Protocol** — before writing any finding,  the target document and the source being audited against. Quote exact text for every specific claim. If you cannot paste the line that proves it, do not write the claim.

**Search Before You Answer** — explicit tool call before any answer about past work, decisions, or project state. Offering to search is not searching.

**Multi-step Isolation** — any task with 5+ steps or 3+ file writes runs as an isolated session with full tool access and self-destructs after completion. Keeps the main session context clean.

**Memory Integrity** — never confirm what cannot be verified. Tool call first, claim second.

---

## Key Design Decisions

**Why local-only inference?**
Privacy and latency. All data stays on-device. No API costs, no rate limits, no data leaving the machine.

**Why two agents instead of one?**
Cognitive specialization. Gemma 4 handles execution and memory tasks well at 12B scale. DeepSeek-R1 is a reasoning model optimized for structured analysis. Running both lets the system match the model to the task instead of forcing one model to do everything.

**Why dual memory?**
Vector recall is great for fuzzy, semantic retrieval. Wiki injection is better for structured reference material — books, source maps, project docs. Neither alone covers the full retrieval surface. MEMORY.md is the curated layer above both — the distilled truth that makes searches targeted instead of blind.

**Why a dreaming system?**
Auto-captured LanceDB memories are high-volume, low-curation. MEMORY.md is low-volume, high-curation. Something needs to promote the best short-term captures to long-term storage and prune what is stale. The nightly pipeline does that without manual intervention.

**Why behavioral protocols instead of just prompting?**
The Commit Protocol exists because a well-prompted agent will still spiral into re-evaluation loops under competing instructions. The Document Review Protocol exists because agents hallucinate audit findings. Hard rules close gaps that soft instructions leave open.

**Why verify before reporting?**
Agents hallucinate. They will claim to have read a file they have not, written content they did not, or remember something that was never stored. Every state claim in this system requires a tool call to verify. "I read it" is not the same as the read tool returning the content.

**Why compaction instead of just a larger context window?**
A 220K token context window sounds like enough — until you have watched a session drift after 100+ messages. Compaction keeps the active context tight and high-signal. Precision over raw capacity.

---

## Autonomy Roadmap

Five items that bridge the gap between reactive assistant and fully autonomous agent, in build order:

1. **Persistent Goals** — a  Maria owns and maintains. Without goals the heartbeat has nothing to execute against.
2. **Heartbeat that executes** — upgrade from vitals check to work loop: read goals → pick next step → run it → log → update status.
3. **Environmental triggers** — webhooks, file watchers, sensors. Pull model (heartbeat) + push model (triggers) covers the full event space.
4. **Self-evaluation loop** — act → verify immediately → re-verify after time passes → flag drift. Closes the one-shot bet problem.
5. **Outcome judgment** — effort budget rule: if N attempts without convergence, stop and escalate. The middle gear between complete and fail.

---

## Status

Active development. Maria is a live production environment — the system I use to work on AI infrastructure problems every day. This documentation reflects architectural decisions and the reasoning behind them.
