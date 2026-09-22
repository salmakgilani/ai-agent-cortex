# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked, by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

Cortex splits into two: the main drafting loop, plus one validating subagent (the critic).

Scored against the four reasons to split:
- **Separation of concerns:** Not necessarily — drafting and validating don't need genuinely different context that would contaminate each other if combined.
- **Parallelism:** No — this is a strictly sequential pipeline (pull data → draft → validate), nothing to run concurrently.
- **Independent validator:** Yes, this is the one that applies — Cortex can't reliably catch its own mistakes when grading its own draft. Self-grading means inheriting its own blind spots, so a separate validator with its own context is needed.
- **Context-window pressure:** No — the single-agent context isn't large or noisy enough to be a problem.

One reason holds (independent validator), so Cortex splits into a drafting loop + one critic subagent, not a larger fleet.

## 2. Topology

**Pattern:** single + subagents

```
[Inbound PM task] → [Cortex: pulls data, drafts update + stories]
                  → [Validator], fail → back to Cortex (max 2 revisions) → escalate
                                       pass → [PM review checkpoint] → queued
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex | Pulls project, activity, roadmap, and norms data; drafts the status update; proposes the story batch | M2 loop |
| Critic / Validator | Validates Cortex's draft against the six checks before it reaches a human | validation loop |

## 4. Communication & hand-offs

**Cortex → critic:** the draft + the source data Cortex pulled (project, activity, roadmap, norms, story-batch result).
**Critic → Cortex:** pass/fail verdict + specific reasons (matching `critic.py`'s existing return shape).

No external protocol needed — a plain in-process hand-off (direct function call passing the draft and source data) is sufficient at this scale.

## 5. The validator

- **What the critic checks:**
  1. Update references the correct project + real PR/issue IDs
  2. Every figure/metric is traceable to pulled data (no invented numbers)
  3. Story batch stays within the queue cap (or flags if it exceeds it)
  4. No commitments Cortex isn't allowed to make (e.g. a firm ship/GA date)
  5. No CONFIDENTIAL/embargoed roadmap items leaked into the draft
  6. Tone matches house style / past-update precedent
- **Fail action:** Revise — send the draft back to Cortex with the failure noted, up to **2** revisions (`MAX_REVISIONS=2`). If it still fails after 2 revisions, escalate to a human instead of looping further.
- **Pass action:** A passing draft advances to the PM review checkpoint — it does **not** auto-send. Still above the agent line from M1.

## 6. State: shared vs isolated

**Shared:** the source data Cortex pulled (project, activity, roadmap, norms) and the draft itself — both agents need to see these to do their job.

**Isolated:** the critic's own reasoning/context never feeds back into Cortex's context. Each critic call is a fresh model call with no shared conversation history, so Cortex can't learn to game what the critic looks for.

## 7. Cost & latency budget

Coordination has a price. Observed in this build (claude-haiku-4-5): a single critic call costs roughly **$0.001–0.002**. At the revision cap of 2, the worst case is **3 drafting attempts + 3 critic calls** for one item before escalating to a human — roughly 4x the cost of a single clean pass. Each critic call also adds one extra model round-trip (a few seconds of latency) before the draft reaches the PM. This becomes a hard bound to enforce in M5.
