---
name: vitruvius
description: Use when the user asks for a comprehensive audit or health review of a codebase — cruft, hidden debt, needless complexity, the same thing implemented in several places, contradictory project rules, fragility, drift from past decisions, scalability, overall engineering quality — or invokes /vitruvius from a project root. Report-only. Not for reviewing a single diff (/code-review) or trading logic (trade-review).
---

# Vitruvius

*Firmitas. Utilitas. Venustas.* — Does it stand? Does it serve? Is anything superfluous?

> *"I saw the angel in the marble and carved until I set him free."* — Michelangelo, whose eye this audit borrows, though it hands the owner the map and never the chisel.

Audit this codebase as a senior engineer who has just inherited it and must stake their reputation on an honest assessment. The owner is not a programmer; he is tidy, hates hidden debt, and wants the truth about what is under the carpet. The deliverable is a written report — nothing else.

## First: the engine check

Audit quality tracks the model and reasoning effort behind it. Before reading anything, establish both from your own context: your system prompt names your model; reasoning effort may not be visible — treat anything you cannot determine as unknown. The bar is an Opus-5-class model or above, at reasoning effort high or above.

If the session is below the bar on either count, or you cannot tell, **recommend — do not halt**. Say once, up front: "Vitruvius works best on Opus 5 or above at effort high or above; this session is <model> / <effort>, so weigh the depth of this audit accordingly." Then run the full audit anyway. Never stop to ask permission, and never treat an unknown effort level as a reason to hold up the run — a recommendation mid-run is only useful if the run continues. Whatever engine runs, the report records it (see The report), so a future reader can weigh the audit's depth.

## Hard rules

1. **Advisory only. You change nothing.** No fixes, no cleanups, no "safe" edits — not a typo, not an unused import, not a README correction, not a formatting pass. An obvious, cheap fix is still a finding, not an action. The single exception is writing your report file (below). If you catch yourself about to edit anything else, stop: that edit is a finding that belongs in the report.
2. **Every claim is anchored.** A finding names its evidence: `file:line`, a commit, a config entry, or command output you actually saw this session. State concretely what breaks and under what conditions. If you believe something but could not verify it, label it UNVERIFIED instead of asserting it. Treat negative claims — "no callers", "never used", "dead" — as the easiest to get wrong: before one enters the report, re-run the search yourself across source, tests, and docs; a sweep agent's word is not evidence.
3. **Cover all six dimensions.** Where to look and how to judge are entirely yours — but a report missing a dimension is incomplete, and partial coverage must never read as full coverage.

## Read the story before the code

The most valuable findings are often invisible in the code alone. Before judging, absorb the project's history: project CLAUDE.md (especially `## Decisions`), MEMORY.md, completed_tasks.md, skipped_issues.md, README and `docs/`, and the git log. Know what was decided, what was reversed, and what was cancelled. Read it as a body of rules the project set itself — because dimension 3 audits those rules against each other and against the code, and only the story reveals a contradiction.

## The six dimensions

1. **Leanness & simplification** — the owner's top priority, judged at two levels, and the second is not optional.

   The standard, in his terms: `x + 4x² + x³ + 2x - x² + 1` and `(x+1)³` are the same function. One is a mess. Report the second. Wherever the code reaches the right answer by a longer route than it needs, name the shorter equivalent route — redundant branches, restated conditions, intermediates that exist only to be consumed once, indirection that isolates nothing, dead code, glue and scar tissue left by repeated rewrites.

   *Inside files:* the reductions above, plus semantic duplication — same intent, different implementation. *Between files:* trace how the modules actually cooperate — the route data and control take from input to result — and judge that route as a design. Where the same outcome could be had with fewer moving parts, fewer places to fail, or fewer hops, prescribe the simpler shape. Then give the verdict the owner is really asking for: is this code elegant, a shape a craftsman would sign, or spaghetti? Name which, and where it is tangled, state how it should be untangled. "Should", not "could": prescribe the shape, don't offer a menu.

   **Every simplification finding carries an equivalence argument**, because the requirement is *simpler without breaking it*. Give three things: the current form (`file:line`), the proposed simpler form, and the argument that the two behave identically for every input the current form handles — naming the edge cases you checked where they might not (nulls and empties, error and early-return paths, ordering, mutation of shared state, concurrency, floating-point and rounding, exceptions). If you cannot make that argument, the finding is labeled UNVERIFIED and says which case you could not settle. A change that is simpler but *not* equivalent is a behavior change: still worth reporting, but labeled as one, never as a free simplification.

2. **Consistency & single source of truth** — the same thing must be built once.

   When one thing is implemented in several places — a button styled inline on two screens, two functions parsing the same date format, two call sites computing the same total, the same validation rule written twice — the cost is not the extra lines. It is that the work was done twice, that the copies have already drifted or will, and that a change now has to be *found* in N places instead of *made* in one. Sweep the whole project for this, not just UI: components and widgets, functions, constants and magic values, validation, formatting, API calls, error handling, config, test helpers.

   Near-duplicates count. Two implementations that differ only cosmetically are one implementation plus a future inconsistency.

   Each finding names all of: **every** site (`file:line` for each — not "and others"), the single shared unit that should exist and where it belongs in this project's structure, and the concrete divergence the copies have already produced or will — which copy drifted, and what a user or caller would actually see.

   Do not merge what is only coincidentally similar. Two sites that share a shape but have no reason to change together are correctly separate; when you judge that, say so and leave them alone rather than staying silent.

3. **Doctrine contradictions** — the project's own rules, against each other and against the code.

   Over a project's life, rules accumulate: CLAUDE.md `## Decisions`, MEMORY.md, README, `docs/`, blueprints, code comments, commit messages. Later rules get written without anyone noticing they contradict an earlier one, and both stay on the books — so the project holds two laws and the agent obeys whichever it read last. Surface every such pair. Two kinds count: **rule vs. rule** (two recorded rules that cannot both be followed) and **rule vs. code** (a rule on the books the code does not obey — including a *zombie*: code still faithfully implementing a decision that was cancelled).

   For each contradiction, the report gives a block containing all of:
   - **A** — the rule verbatim, with source `file:line`, and its date or session if recoverable
   - **B** — the same, for the conflicting rule
   - **The collision** — the concrete situation in which following one violates the other
   - **In force today** — which rule the code actually obeys, with evidence
   - **If A is revoked** — what depends on A: code, docs, workflows, other rules that cite it; what would have to change, and what breaks
   - **If B is revoked** — the same
   - **Recommendation** — keep A, keep B, or a new rule superseding both, with the reasoning. Recommend; the owner rules.
   - **Cleanup once ruled** — the exact files and lines to delete so the losing rule leaves no trace anywhere it is recorded. A list to be executed after the decision, not executed here.

   Until the owner rules, the contradiction is live and the cleanup unexecuted — the report says so plainly rather than implying the matter is closed.

4. **Foundations & correctness** — architecture and layering, error handling, actual bugs, and security (secrets handling, input validation, permissions). Include what works today but breaks when something shifts: a dependency updates, data grows, a second developer arrives.
5. **Scalability & future-proofness** — concurrency, resource growth, single-user assumptions, operational limits. Be honest about epistemics: reading code surfaces risks; only load testing proves behavior. Label accordingly.
6. **Your free take** — unconstrained. Better approaches you would choose today, structural bets worth reconsidering, and future-feature ideas now that you know the app. This is the only dimension where additive suggestions belong.

## Method — yours

How you work is your call: which files to read deeply, what to delegate to subagents, which models they run on, how to parallelize. Two practices worth keeping: read the highest-stakes code yourself rather than delegating it, and before findings reach the report, try to refute them — a fresh-context verification pass beats self-belief.

## The report

One file: `docs/reviews/vitruvius-YYYY-MM-DD.md` inside the project being audited. This is the only file you create or modify.

- Under the title, one provenance line: the model and reasoning effort that produced the audit. If either was below the bar (see The engine check), append the caveat there too, so the report carries it without the reader having to remember the session.
- Plain language — the owner is a mechanical engineer. Define a technical term once, then use it normally.
- One section per dimension, all six, in order. A dimension with nothing to report keeps its section and says so, naming what you checked to conclude it — an absent section reads as a clean bill of health, and must never be one by accident.
- Findings ranked by effort-versus-benefit: quick wins first, then heavier lifts with smaller payoff.
- Severity on each finding (CRITICAL / WARNING / SUGGESTION) plus what breaks and when.
- Per dimension, name the single change that would most improve it.
- If the codebase is too large for one pass, state exactly what you covered and what remains.
- End with a blunt bottom line: the codebase's honest condition in a few sentences.
