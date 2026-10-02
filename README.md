# Cortex, a PM Chief-of-Staff Agent

> A PM chief-of-staff agent with an independent critic: it triages a PM task, pulls internal state, and preps a status update plus a story batch, so the team approves instead of assembling from scratch.

_Salma K · Agentic Loops for PMs Cohort · Oct 2026_

Repo: https://github.com/salmakgilani/ai-agent-cortex

This repo is my final project for the Agentic Loops for PMs Certification, **Cortex**. Each module’s artifact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artifacts

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration &amp; Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Context Engineering &amp; Memory
- **Memory &amp; context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds &amp; Evals
- **Bounds &amp; evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy &amp; Production
- **Production &amp; autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype write-up**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)

---

## Ship plan

### Autonomy dial (per segment)
- Seasoned PM running weekly updates: Supervised (can catch a wrong number, approves quickly)
- New PM / new to the project: Assisted (every draft gets a full review)
- Exec reading the output: Shadow, no direct access (sees only PM-approved output)
The dial changes how many below-the-line actions pause for review; it never moves
the agent line.

### Trust Ladder rung + eval gate
Current rung: Supervised, on mock fixtures only (clean-record window not started).
Gate to bounded-autonomous: >=95% pass on EV-1/EV-2/EV-4 and 100% on EV-3/EV-5 over
4 weeks of supervised real-data runs, with zero trust incidents.
Measured baseline: last 7 happy-path runs, 5 clean and 2 failed task completion
(~71%). Gate not yet met.

### Deployment plan
Runtime: serverless. Owner: the PM who owns Cortex; backup: a fellow PM or squad
engineer who can revoke the API key. Escalations save a held draft and email the
team DL. Rollback: revert prompts/critic to the last tagged commit, remove a tool,
drop the dial a rung, or revoke the API key. Monitoring: eval pass % per case,
escalation rate, cost per run vs $0.50, daily spend vs $5, trust incidents.

### ROI metrics + widen-autonomy rule
Outcome: >=80% of drafts approved without a rewrite.
Cost-to-serve: <=$0.05 per approved draft (observed ~$0.01-0.03 per run).
Trust incidents: 0 confidential leaks, committed dates, unapproved posts, or bound breaches.
WIDEN-AUTONOMY RULE
Move the seasoned-PM segment to bounded-autonomous only after 4 consecutive weeks
meeting the gate, >=80% no-rewrite approvals, and zero trust incidents. Any incident
resets the clock; the new-PM segment stays at Assisted until then.

### Governance &amp; strategy
Compliance: no PII or credentials in prompts/fixtures; confidential roadmap items are
blocked from output by the critic. Safety: posting, committing dates, and ticket
changes stay above the line for everyone; kill switch is revoking the API key.
Reliability: 8-iteration, $0.50/run, 10-story, and 2-revision caps enforced in code;
the 90s timeout and $5/day cap are specified but not yet implemented; fails closed if
the model is down. Next: a real GitHub connector, gated by the eval suite re-run on
real data

---

## Build insights

- **Friction point.** porting from OpenAI to Claude, Haiku wrapping the critic's JSON in prose, and forcing a bad draft taking three tries because Cortex resisted misbehaving.
- **Key learning.** the human line is enforced by infrastructure, not prompts; the critic is only as good as its rules; a bound is a counter or budget the model can't argue with.
- **Aha moment.** an agent isn't "done" when the demo works, which is why the gate is a number over a window.

---

_Certification submission, Agentic Loops for PMs Certification._
