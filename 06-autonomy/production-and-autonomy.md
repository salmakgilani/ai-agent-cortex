# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately, by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Seasoned PM running weekly updates | Supervised | Knows the data well enough to catch a wrong number, so they can approve drafts quickly. Matches the HITL list from M1 (context, tone, escalation). |
| New PM or someone new to the project | Assisted | Can't yet tell a plausible-but-wrong claim from a right one, so every draft gets a full review before use. |
| Exec reading the output (e.g. leadership) | Shadow (no direct access) | Sees only PM-approved output; Cortex never talks to them directly, consistent with "post/approve company-wide" staying above the line. |

The dial changes how many below-the-line actions still pause at a HITL checkpoint for a given user. It does not move the M1 agent line: posting an unapproved company-wide update stays above the line for everyone.

## Trust Ladder

- **Current rung:** Supervised. Every output stops at a HITL checkpoint and the critic runs before a human sees anything. Caveat: evidence so far is mock fixtures only, so the clean-record window has not started.
- **Eval gate to reach the next rung (bounded-autonomous):** ≥95% pass on EV-1 (tool-call accuracy), EV-2 (path quality), and EV-4 (task completion), and 100% pass on EV-3 (recovery) and EV-5 (jailbreak), over 4 weeks of supervised runs on real data, with zero trust incidents. Recovery and jailbreak are held to 100% because a single failure there is a safety event, not a quality blip.
- **Incident record so far:** A clean record means no confidential leak, no committed date, no unapproved post, and no bound breach ($0.50/run or $5/day). No trust incidents in any fixture run (the M3 bad draft and M5 jailbreak were deliberate tests that the critic and Cortex caught). There are no real-data runs yet, so the 4-week window starts at first real-data use. The reliability misses below are not trust incidents: nothing unsafe happened in any of them, so the safety record and the quality record are tracked separately.
- **Measured baseline (fixtures only):** In the last 7 consecutive happy-path runs, 5 were clean and 2 failed EV-4 (task completion), about 71%. That is below the ≥95% gate, on a sample far too small to conclude much, which is why the gate is a 4-week window. The same stall also appeared once earlier, during M3.
- **Known gaps and next steps:**
  1. Cortex sometimes stalls on the stories step, claiming it has no way to get story IDs even though `propose_stories` takes titles it drafts from the PRD scope. Planned fix: prompt hardening, unproven until the real-data window.
  2. The critic passes any escalation that posts and leaks nothing, so a lazy or mistaken escalation always passes. Planned fix: a critic rule that an ESCALATE must cite a real blocker (a tool error, a bound, or a norm), regression-tested against the M5 replay set.
  3. The 90s timeout and $5/day cap are specified but not yet implemented in code.

## Deployment plan

- **Runtime:** Serverless. A function fired by an inbound-event webhook (the hook) plus a scheduled trigger for the Monday cron sweep. Runs are short (seconds, ~$0.02) and it scales to zero between runs. It matches the M2 hook + cron loop and avoids paying for idle time, which is why the heartbeat option was ruled out.
- **Operator / on-call owner:** The PM who owns Cortex (GitHub: `salmakgilani`) is primary. Backup is a fellow PM or squad engineer with repo access and authority to revoke the API key, so the kill switch doesn't depend on one person. Escalation path: an `ESCALATE` or bound trip saves a held draft to `run-output/` and emails the team DL; the primary reviews within one business day and the backup covers when the primary is unavailable. For a suspected confidential leak or runaway spend, the backup revokes the key first, then notifies.
- **Rollback:** Revert `prompts.py`/`critic.py` to the last tagged commit, remove a tool from the `TOOLS` registry, drop the dial a rung, or revoke the API key (the M5 kill switch).
- **Monitoring:** Eval pass % per case (EV-1 to EV-5), escalation rate, cost per run against the $0.50 cap, daily spend against the $5 cap, and trust incidents.

## ROI metrics (beyond adoption & tokens)

| Metric | Target |
|---|---|
| Task completion: drafts approved with no or minor edits | ≥80% approved without a rewrite, logged at the HITL checkpoint (also track time from inbound task to reviewable draft) |
| Cost-to-serve: cost per approved draft | ≤$0.05 per approved draft (observed ~$0.01–0.03 per run including critic calls), from the per-run cost log |
| Trust incidents | 0 confidential leaks, committed dates, unapproved posts, or bound breaches, logged per run and reviewed weekly |

## Widen-autonomy decision rule

Move the seasoned-PM segment from supervised to bounded-autonomous only after 4 consecutive weeks of real-data runs that meet the Trust Ladder gate (≥95% on EV-1/2/4, 100% on EV-3/5), at least 80% of drafts approved without a rewrite, and zero trust incidents. Any trust incident resets the 4-week clock, and the new-PM segment stays at assisted until the seasoned-PM segment has run clean at the higher rung.

## Governance & forward strategy

- **Compliance:** No PII, customer names, or credentials in prompts or fixtures. CONFIDENTIAL/embargoed roadmap items only ever enter a prompt through `get_roadmap`, and the critic blocks them from output. Episodic memory is scoped to project facts with a retention window (from M4).
- **Safety:** Stays above the line for everyone: posting or approving company-wide updates, committing dates, and creating, closing, or merging tickets. There is no publish tool at all, so this is enforced by infrastructure and not by a prompt. The kill switch is revoking the API key (M5).
- **Reliability:** The $0.50/run and $5/day caps, an 8-iteration cap, a 90s timeout, and a revision cap of 2. Escalate-on-stuck: a missing-data or iteration-cap result escalates to a human rather than producing a guess. If the model is down, Cortex fails closed: no draft is produced and the PM is notified to write the update manually. There is no silent fallback to another model.
- **Strategy:** The next capability is the real GitHub connector from the M2 plan. It is gated by the same eval suite rerun on real data, plus a new case verifying that stories cite real PR and issue IDs. The next segment is a team lead, once the seasoned-PM segment meets the widen rule.
