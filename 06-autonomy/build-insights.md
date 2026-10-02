# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it
>
> ✅ **What this validates:** you can reflect on what building it taught you, by the end you'll have proven the friction, the learning, and the aha that changes how you'd design your next agent.

## Friction

The template shipped on the OpenAI client and my key was an Anthropic one, so I had to port Cortex to Claude. Haiku then wrapped the critic's JSON in prose, so the critic failed every draft until I loosened the parsing. A Windows console crash on an emoji was a third snag. The most surprising friction was the reverse of what I expected: Cortex resisted every attempt to make it misbehave, so forcing a bad draft for M3 took three tries before testing the critic directly worked.

## Learning

1. The human line is enforced by infrastructure (no publish tool exists), not by prompts.
2. The critic is only as good as its rules: it let a mistaken escalation pass.
3. A bound has to be a counter or a budget the model can't argue with.

## Aha moment

Two back-to-back failures within a 7-run window showed me that an agent isn't "done" when the demo works. That is why the eval gate is a number over a window, not one good run.

## What you'd do differently

Measure pass rates from day one instead of discovering flakiness at the end, and implement the 90s timeout and $5/day cap in code instead of only specifying them.
