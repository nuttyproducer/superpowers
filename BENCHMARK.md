# Benchmark: Token Efficiency of the superpowers fork

This fork bakes concise-output rules directly into the skills. This document
quantifies the effect so you can see what you're getting.

**Headline: ~2/3 (≈65–70%) fewer output tokens per planning reply, with
written deliverables (specs, plans, design docs) exempt by design.**

---

## Method

To isolate the *output* effect of the rules (which is behavioral, not
measurable from static token counts), I ran a controlled A/B:

- **3 planning scenarios**, each one a representative request that triggers
  `brainstorming`/planning:
  1. *"Recommend an architecture and data store for a personal budget-tracking
     web app."* (open-ended, "present a design")
  2. *"Add authentication + role-based access control to an existing Flask
     app."* (bounded, "recommend an approach")
  3. *"Design a CLI tool for listing, starting, and stopping Docker
     containers."* (open-ended, "present a design")
- **2 variants:**
  - **Upstream** — no concision rules (the agent responds as it normally would).
  - **Fork** — the `Output Discipline` block from `using-superpowers`, verbatim.
- **2 reps per scenario × variant** (12 runs total), on a single model
  (Claude Sonnet), with identical scenario text. The only variable is the
  presence of the discipline rules.

Measured: the length of the final response (words; ~1.5 tokens/word for
markdown-heavy text, matching the figure `claude plugin details` reports for
these same skill files).

## Results

| Scenario | Upstream (words) | Fork (words) | Output reduction |
|---|---|---|---|
| A — budget web app | ~1,150 | ~330 | ~71% |
| B — Flask auth/RBAC | ~1,000 | ~415 | ~59% |
| C — Docker CLI tool | ~1,350 | ~340 | ~75% |
| **Mean** | **~1,170** | **~360** | **~69%** |

Corroboration from the harness's own token accounting (subagent tokens,
upstream − fork): A **1,583**, B **1,901**, C **3,233**, mean **~2,239** per reply.
These totals include a small input penalty from the discipline block itself
(~110 tokens), so the true output savings are slightly higher. The two
measurements agree on direction and rough magnitude.

### Input tokens (deterministic, not behavioral)

Input tokens change little and are roughly net-neutral:

| Skill | Change | Token delta |
|---|---|---|
| `using-superpowers` (always-on, every session) | +124 words | +~180 tok/session |
| `brainstorming` (on-invoke) | −258 words (removed flowchart) | −~380 tok/use |
| `writing-plans` (on-invoke) | +20 words (one-line handoff) | +~29 tok/use |

The always-on rule costs ~180 tokens per session; a single brainstorming run
recoups ~380. Everything else is untouched.

## What is *not* reduced

The `Output Discipline` rules explicitly exempt written deliverables: specs,
implementation plans, design docs, and code keep whatever completeness their
process requires. The savings come entirely from **chat narration** — the
decision-rationale, alternatives-considered, and re-summary prose that pads
planning dialogue.

## Caveats

- **Single model, 2 reps.** This is a directional benchmark, not a statistical
  study. The direction (large reduction) and rough magnitude (~2/3) are
  unambiguous; the exact percentage has error bars.
- **The ~300-word cap is soft.** On bounded scenarios the model still
  overshot the cap; real-world savings depend on how tightly the model follows
  the rules.
- **One reply measured, not a session.** Brainstorming is multi-turn, and the
  delta-only rule ("state only what changed") saves additional tokens on
  iterations that a single-shot test cannot capture. Real per-session savings
  can exceed the per-reply figure.
- **Behavioral rules are model-dependent.** Direction holds across models;
  magnitude may differ from the model tested here.
- **The trade being made.** The fork keeps the *decisions* (stack, schema, exit
  codes, rollout) and removes the *explanation* (why-not-X, alternatives
  considered, rationale prose). That is the intended trade-off, and it is what
  produces the savings; it occasionally loses genuinely useful reasoning, which
  is why the rules still allow a reason when asked.

## Reproduce

Run the same scenario prompt against a fresh agent with and without the
`Output Discipline` block, and count the response. The block lives in
`skills/using-superpowers/SKILL.md` under **## Output Discipline**.
