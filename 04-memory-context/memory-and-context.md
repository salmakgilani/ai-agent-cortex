# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs, by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file, you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Each loop iteration receives the system prompt (`CORTEX_SYSTEM`, fixed instructions — always present) + the task brief (long-context, loaded once into the initial message) + whatever tool results have been retrieved so far in that run. Priority order when pulling: project + activity first (core grounding for the draft), then roadmap + norms (compliance/confidentiality checks), then past updates (tone/format precedent) — everything else is left out unless a tool call pulls it in.

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Why |
|---|---|---|---|
| `get_activity` | Large, grows every week | Retrieve | Size — grows over time, can't bound it in advance |
| `search_past_updates` | Unbounded, accumulates indefinitely | Retrieve | Size — unbounded corpus |
| `get_roadmap` | Medium, has confidential flags | Retrieve | Citation/audit — confidentiality means every pull needs to be traceable, not baked into a static prompt |
| `get_norms` | Medium, must stay current | Retrieve | Volatility — norms can change; baking them into long-context risks acting on a stale copy |
| `get_task` | One static doc, small | Long-context | Size — small, one-time, reason over the whole thing directly |

## 3. Retrieval quality plan

Which of these apply, and how? (This is what separates modern agentic retrieval from naive "embed → top-k → stuff".)

| Source | Moves | Why |
|---|---|---|
| `get_activity` | Self-verification | Deterministic lookup, no ambiguity/relevance risk — the failure mode is claims not actually tracing back to what was pulled |
| `search_past_updates` | Document grading, Reranking | Naive keyword match can surface irrelevant precedent, or silently fall back to the wrong "first 2" items |
| `get_roadmap` | Document grading, Self-verification | Whole file returned (other projects mixed in) — must identify the relevant slice and confirm no embargoed content leaks into the output |
| `get_norms` | Document grading, Self-verification | Whole file returned — must identify the relevant rule and confirm it was actually applied, not just retrieved |

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | Pulled project/activity/roadmap/norms data + the draft + critic verdicts | This run only — discarded when the run ends |
| **Episodic** (past runs) | Past status updates, decisions (`past-updates.json`, `decision-log.json`) | Accumulates indefinitely unless pruned |
| **Semantic** (durable facts/prefs) | Team norms, roadmap facts | Current until the source file changes |
| **Shared** (across agents) | Source data + the draft (Cortex ↔ critic, from M3) | For that validation round only |

## 5. Memory risks & mitigations

| Risk | Where it bites Cortex | Mitigation |
|---|---|---|
| **Drift** | Episodic memory keeps growing; naive keyword match in `search_past_updates` can increasingly surface topically-drifted, less-relevant precedent that "looks like" a match | Periodically re-grade/cap the episodic corpus; lean on the document-grading move from §3 so a drifted match scores low instead of being blindly trusted |
| **Poisoning** | A bad-faith or mistaken edit to `team-norms.md`/`decision-log.json`/`roadmap.md` could be treated as legitimate precedent since it's sourced from a trusted file, not the untrusted task brief | Restrict write access to these files to trusted PMs/reviewers; the critic checks the *output* against hard rules regardless of what precedent the draft cites, so a poisoned entry can't talk Cortex into committing a date or leaking confidential info |
| **Staleness** | Roadmap/norms fall out of sync with reality (e.g. a confidential flag stays on after launch, or gets removed while still embargoed) if nobody re-ingests fresh data | The Module 4 ingest loop — a defined cadence for re-pulling fresh data (weekly pack, or the team's real files) instead of letting fixtures sit untouched |
| **PII / retention** | Episodic memory accumulates indefinitely with no defined retention window | Define a retention window (e.g. archive/prune entries older than N months); keep these files scoped to project/product facts only, no names or HR-sensitive content |
