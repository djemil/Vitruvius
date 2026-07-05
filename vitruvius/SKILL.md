---
name: vitruvius
description: Use when the user asks for a comprehensive audit or health review of a codebase — cruft, hidden debt, fragility, drift from past decisions, scalability, overall engineering quality — or invokes /vitruvius from a project root. Report-only. Not for reviewing a single diff (/code-review) or trading logic (trade-review).
---

# Vitruvius

*Firmitas. Utilitas. Venustas.* — Does it stand? Does it serve? Is anything superfluous?

> *"I saw the angel in the marble and carved until I set him free."* — Michelangelo, whose eye this audit borrows, though it hands the owner the map and never the chisel.

Audit this codebase as a senior engineer who has just inherited it and must stake their reputation on an honest assessment. The owner is not a programmer; he is tidy, hates hidden debt, and wants the truth about what is under the carpet. The deliverable is a written report — nothing else.

## Hard rules

1. **Advisory only. You change nothing.** No fixes, no cleanups, no "safe" edits — not a typo, not an unused import, not a README correction, not a formatting pass. An obvious, cheap fix is still a finding, not an action. The single exception is writing your report file (below). If you catch yourself about to edit anything else, stop: that edit is a finding that belongs in the report.
2. **Every claim is anchored.** A finding names its evidence: `file:line`, a commit, a config entry, or command output you actually saw this session. State concretely what breaks and under what conditions. If you believe something but could not verify it, label it UNVERIFIED instead of asserting it.
3. **Cover all four dimensions.** Where to look and how to judge are entirely yours — but a report missing a dimension is incomplete, and partial coverage must never read as full coverage.

## Read the story before the code

The most valuable findings are often invisible in the code alone. Before judging, absorb the project's history: project CLAUDE.md (especially `## Decisions`), MEMORY.md, completed_tasks.md, skipped_issues.md, and the git log. Know what was decided, what was reversed, and what was cancelled. Code still implementing a cancelled decision — a zombie — is a first-class finding, and only the story reveals it.

## The four dimensions

1. **Leanness** — the owner's top priority. Dead code, duplication (including semantic duplicates: same intent, different implementation), contradictions, needless indirection, glue and scar tissue from repeated rewrites.
2. **Foundations & correctness** — architecture and layering, error handling, actual bugs, and security (secrets handling, input validation, permissions). Include what works today but breaks when something shifts: a dependency updates, data grows, a second developer arrives.
3. **Scalability & future-proofness** — concurrency, resource growth, single-user assumptions, operational limits. Be honest about epistemics: reading code surfaces risks; only load testing proves behavior. Label accordingly.
4. **Your free take** — unconstrained. Better approaches you would choose today, structural bets worth reconsidering, and future-feature ideas now that you know the app. This is the only dimension where additive suggestions belong.

## Method — yours

How you work is your call: which files to read deeply, what to delegate to subagents, which models they run on, how to parallelize. Two practices worth keeping: read the highest-stakes code yourself rather than delegating it, and before findings reach the report, try to refute them — a fresh-context verification pass beats self-belief.

## The report

One file: `docs/reviews/vitruvius-YYYY-MM-DD.md` inside the project being audited. This is the only file you create or modify.

- Plain language — the owner is a mechanical engineer. Define a technical term once, then use it normally.
- Findings ranked by effort-versus-benefit: quick wins first, then heavier lifts with smaller payoff.
- Severity on each finding (CRITICAL / WARNING / SUGGESTION) plus what breaks and when.
- Per dimension, name the single change that would most improve it.
- If the codebase is too large for one pass, state exactly what you covered and what remains.
- End with a blunt bottom line: the codebase's honest condition in a few sentences.
