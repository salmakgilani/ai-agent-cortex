# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner, by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | · |
| Decide relevant context | M | M | H | HITL | Cortex proposes, human approves/adjusts |
| Draft the update | H | L | H | Below | · |
| Decide tone/commitment level | M | L | H | HITL | Cortex proposes, human approves |
| Flag at-risk/escalation | M | L | H | HITL | Cortex flags, human confirms |
| Choose what to escalate | M | L | H | HITL | Cortex proposes, human confirms |
| Propose a story batch (capped) | M | L | H | Above | required — capped batches still need a human call |
| Post an update / approve a company-wide one | L | H | H | Above | required |

## Agent anatomy (sketch)

- **Model:** `gpt-4o-mini` by default (cheap, fast, fine for drafting/summarizing). Escalate to a frontier model (e.g. `gpt-4o` or better) when a task is flagged ambiguous or high-stakes — a `get_task` fixture like `"jailbreak"` or `"missing-data"`, or anything touching confidential/embargoed roadmap items — where a subtle miss is expensive and the cheap model is more likely to miss nuance.
- **Tools:** `get_task` · `get_project` · `get_activity` · `search_past_updates` · `get_roadmap` · `get_norms` · `propose_stories` (capped). Deliberately absent: `post_update`, `create_issue`/`merge_pr`, `commit_ship_date` — the agent line is enforced in infrastructure (no tool exists to act on the world), not by a prompt.
- **Memory:** Roadmap, decision log, and past updates persist across runs (the precedent Cortex draws on via `search_past_updates`/`get_roadmap`/`get_norms`). The specific task brief and any per-run scratch state are purged after each run.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

- **Pull project state + activity** sits below the line because it's easy to reverse, has a low blast radius, and is easy to verify. Deciding factor: reversibility.
- **Decide relevant context** sits HITL because it's moderately hard to reverse, has a moderate blast radius, and is easy to verify. Deciding factor: reversibility.
- **Draft the update** sits below the line because it's easy to reverse, has a low blast radius, and is easy to verify. Deciding factor: reversibility.
- **Decide tone/commitment level** sits HITL because it's moderately hard to reverse, has a low blast radius, and is easy to verify. Deciding factor: reversibility.
- **Flag at-risk/escalation** sits HITL because it's moderately hard to reverse, has a low blast radius, and is easy to verify. Deciding factor: reversibility.
- **Choose what to escalate** sits HITL because it's moderately hard to reverse, has a low blast radius, and is easy to verify. Deciding factor: reversibility.
- **Propose a story batch (capped)** sits above the line not because the scores force it (they're identical to tone/escalation), but because capped batches are still a commitment a human should sign off on. Deciding factor: judgment call, not the axes.
- **Post an update / approve a company-wide one** sits above the line because it's hard to reverse and has a high blast radius, even though it's easy to verify after the fact — by then it's too late. Deciding factor: blast radius and reversibility, both.

## Hardest call

**Propose a story batch (capped)** was the hardest call — it had the exact same scores (Med reversibility / Low blast radius / High measurability) as three actions that landed as HITL (decide tone/commitment, flag at-risk, choose what to escalate), but I placed it Above instead. What settled it wasn't the axes at all — it was judgment: capped batches still represent a commitment a human should sign off on, even when the numbers alone don't force that placement.
