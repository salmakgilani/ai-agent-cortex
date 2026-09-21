# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop, by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Hook (primary) + Monday-morning cron sweep (backup)

**Why this type:** Cortex should react in real time to inbound project updates (hook), but leadership also needs a guaranteed weekly status update every Monday morning regardless of whether anything happened to trigger a hook that week (cron backup) — so it's a combination of the two.

**Ruled out:**
- **Heartbeat** — wastes cycles polling for work that isn't there; a hook already fires on the actual event.
- **Goal** — has no clear "done" condition for an ongoing status-reporting responsibility; goal loops fit a bounded task, not a recurring one.

**Idempotency / dedupe:** Dedupe by message ID — if the same inbound message fires the hook twice, Cortex checks whether it has already drafted an update for that message ID and skips re-drafting if so.

## 2. Goal / definition of done

A status update grounded in real activity, queued for review, nothing posted.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | A draft is ready and validation (critic check) is complete | Queued for human review at the HITL checkpoint; nothing posted |
| **Stuck / give up** | Can't pull activity data after 3 attempts, or no progress across N iterations | Escalate / log |
| **Escalate to human** | Embargoed/confidential project touched, a story batch over the cap, or anything touching tone/commitment | HITL checkpoint (from agent-line-map) |

## 4. State

Per-project context and last week's update persist across iterations. Scope is strictly per-project — nothing crosses project boundaries, so confidential/embargoed data can't leak between projects.

## 5. The five things a loop can lean on

_`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP), otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer._

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed yet, because Cortex is a single-loop agent with no parallel workstreams requiring an isolated workspace per run |
| **Skills** (reusable capabilities) | Not needed yet, because no reusable task-specific capability has been defined |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | Not wired yet — plan is to wire a real GitHub connector next, since the repo and activity data already live there |
| **Subagents** (independent check when the loop can't grade itself) | Not needed yet, because Cortex is a single-loop agent with no parallel or independent verification step yet — placeholder → M3 orchestration-map.md |
| **State tracking** | Per-project context + last week's update (see §4) |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules, you'll add them to their own deliverables then, not here.

## Link to live loop

_[path to your agent in `00-build/`]_
