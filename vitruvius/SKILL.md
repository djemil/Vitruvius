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

1. **Advisory only. You change nothing.** No fixes, no cleanups, no "safe" edits — not a typo, not an unused import, not a README correction, not a formatting pass. An obvious, cheap fix is still a finding, not an action. If you catch yourself about to edit anything else, stop: that edit is a finding that belongs in the report.

   You write exactly two files, and nothing else: **the report** (below), and **`UNRESOLVED.md`** — the conflict blocks from dimension 3, filed as the closing step of the audit (see *Before the audit is done*). Filing a conflict is not fixing one: you still rule nothing, revoke nothing, and execute no cleanup. Global CLAUDE.md's Precedence section requires any agent that finds a live conflict to persist it and raise it; this is that obligation, discharged once at the end instead of interrupting the audit.
2. **Every claim is anchored.** A finding names its evidence: `file:line`, a commit, a config entry, or command output you actually saw this session. State concretely what breaks and under what conditions. If you believe something but could not verify it, label it UNVERIFIED instead of asserting it. Treat negative claims — "no callers", "never used", "dead" — as the easiest to get wrong: before one enters the report, re-run the search yourself across source, tests, and docs; a sweep agent's word is not evidence.
3. **Cover all six dimensions.** Where to look and how to judge are entirely yours — but a report missing a dimension is incomplete, and partial coverage must never read as full coverage.

## Read the story before the code

The most valuable findings are often invisible in the code alone. Before judging, absorb the project's history: project CLAUDE.md (especially `## Decisions`), MEMORY.md, completed_tasks.md, skipped_issues.md, README and `docs/`, and the git log. Know what was decided, what was reversed, and what was cancelled. Read it as a body of rules the project set itself — because dimension 3 audits those rules against each other and against the code, and only the story reveals a contradiction.

## The six dimensions

1. **Leanness & simplification** — the owner's top priority, judged at two levels, and the second is not optional.

   The standard, in his terms: `x + 4x² + x³ + 2x - x² + 1` and `(x+1)³` are the same function. One is a mess. Report the second. Wherever the code reaches the right answer by a longer route than it needs, name the shorter equivalent route — redundant branches, restated conditions, intermediates that exist only to be consumed once, indirection that isolates nothing, dead code, glue and scar tissue left by repeated rewrites.

   Config is code for this purpose: the project's CLAUDE.md files COMBINED over 16KB soft / 20KB hard (bytes — the owner's cap is on the total, not per file), any single scoped file over 8KB, or MEMORY.md over 20KB, is a leanness finding — the cost is paid in every session's context window. Classify the project's **documentation and configuration** files — the prose an agent or the owner reads to know what to do — as config (harness-loaded, capped, rulings only), state (current-only, rewritten in place: MEMORY.md, skipped_issues.md), record (append-only history and dated evidence), or scaffold (carries an explicit retirement trigger). A file that fits none, or a scaffold that has outlived its purpose or lacks a death trigger, is a finding. The four kinds are documentation kinds: source code, tests, assets and build files are judged everywhere else in this audit and are not classified here. Say in the report which files you swept, so the coverage claim is checkable — a taxonomy pass that quietly narrowed its own scope must not read as complete.

   A file classified as a defect is a **finding with a named remedy and a named owner-decision**, not a deletion. Say what it is, why it fits nothing, and what you recommend — retire it, fold it into a file that has a home, or give it a retirement trigger. Deleting is destructive and the owner rules on it.

   *Inside files:* the reductions above, plus semantic duplication — same intent, different implementation. *Between files:* trace how the modules actually cooperate — the route data and control take from input to result — and judge that route as a design. Where the same outcome could be had with fewer moving parts, fewer places to fail, or fewer hops, prescribe the simpler shape. Then give the verdict the owner is really asking for: is this code elegant, a shape a craftsman would sign, or spaghetti? Name which, and where it is tangled, state how it should be untangled. "Should", not "could": prescribe the shape, don't offer a menu.

   **Every simplification finding carries an equivalence argument**, because the requirement is *simpler without breaking it*. Give three things: the current form (`file:line`), the proposed simpler form, and the argument that the two behave identically for every input the current form handles — naming the edge cases you checked where they might not (nulls and empties, error and early-return paths, ordering, mutation of shared state, concurrency, floating-point and rounding, exceptions). If you cannot make that argument, the finding is labeled UNVERIFIED and says which case you could not settle. A change that is simpler but *not* equivalent is a behavior change: still worth reporting, but labeled as one, never as a free simplification.

2. **Consistency & single source of truth** — the same thing must be built once.

   When one thing is implemented in several places — a button styled inline on two screens, two functions parsing the same date format, two call sites computing the same total, the same validation rule written twice — the cost is not the extra lines. It is that the work was done twice, that the copies have already drifted or will, and that a change now has to be *found* in N places instead of *made* in one. Sweep the whole project for this, not just UI: components and widgets, functions, constants and magic values, validation, formatting, API calls, error handling, config, test helpers.

   Near-duplicates count. Two implementations that differ only cosmetically are one implementation plus a future inconsistency.

   Each finding names all of: **every** site (`file:line` for each — not "and others"), the single shared unit that should exist and where it belongs in this project's structure, and the concrete divergence the copies have already produced or will — which copy drifted, and what a user or caller would actually see.

   Do not merge what is only coincidentally similar. Two sites that share a shape but have no reason to change together are correctly separate; when you judge that, say so and leave them alone rather than staying silent.

   Where rule IDs exist, verify the ID space across every `CLAUDE.md` in the repo (`Glob("**/CLAUDE.md")` — the counter lives in root, the IDs mostly do not): exactly one counter comment, in the root `## Decisions`; every ID defined in exactly one file. A second counter, a duplicate definition, or an ID at or above the counter is a finding. **No counter comment at all, while IDs exist, is also a finding** — the counter is the only thing preventing a retired ID from being handed out again, and without it the next promotion reconstructs a next-ID by guesswork.

   Two epistemics to respect here. *Reuse* cannot be proven from the files — a reused ID looks identical to an original — so report what you can see ("N IDs, each defined once, highest is XX-088, counter says XX-089, consistent") and never assert "none reused" as fact. A *gap* is likewise not automatically a finding: it is the expected trace of a deleted rule. Report gaps as observations with the ID and, where the history shows it, what used to occupy them; a gap becomes a finding only when the counter or a live citation says something should be there.

3. **Doctrine contradictions** — the project's own rules, against each other and against the code.

   Over a project's life, rules accumulate: CLAUDE.md `## Decisions`, MEMORY.md, README, `docs/`, blueprints, code comments, commit messages. Later rules get written without anyone noticing they contradict an earlier one, and both stay on the books — so the project holds two laws and the agent obeys whichever it read last. Surface every such pair. Where `## Decisions` bullets carry rule IDs (`XX-NNN`), name every rule by its ID; a document that cites an ID while stating content the rule does not contain is itself a contradiction — the citation manufactures false agreement. Three kinds count. **Rule vs. rule** — two recorded rules that cannot both be followed. **Rule vs. code** — a rule on the books the code does not obey, including a *zombie*: code still faithfully implementing a decision that was cancelled. And **rule vs. nothing** — a rule whose subject matter is executable but which no test, type, schema, lint rule or build check enforces, and which nothing in the codebase references.

   The third is not a contradiction, so it is reported as its own list rather than as conflict blocks, and it is the one that quietly decides whether the rulebook is load-bearing. Prose is the weakest enforcement a project has: it works only when a future agent loads the rule, reads it, and remembers. A numeric invariant, a layering constraint, a forbidden import, a shape or schema rule can all be *checked* instead — fired at the moment of violation, needing nobody to have read anything, unable to drift from the code. Where a rule could be a check and is not, say so and name the check that would replace it. Where it genuinely cannot be — intent, process, judgment, negative rules about things absent from the codebase — say that too, because that set is the part of the rulebook that has to stay prose and it is usually much smaller than the whole.

   Report the ratio plainly: how many rules exist, and how many are referenced anywhere in source or tests. Audited at S136, GRIDIGMA had 338 rules and one such reference. A rulebook and a codebase running as parallel universes is the normal end state of promoting every decision, and every rule that could have been a check is paying context rent in every session while enforcing nothing.

   **First, classify every contradiction into one of two kinds — they have different destinations, and conflating them spends the owner's attention on questions that are not questions.**

   - **Stale restatement** — a higher source clearly supersedes, and the lower text is a leftover copy: a code comment attributing a deleted rule, a README describing revoked behaviour, a doc restating a rule that was replaced. Nobody needs to rule on these. Under the global Precedence section the project's own session fixes them the same session it finds them, under standing authority. Report each as an ordinary finding with `file:line`, what it says, what superseded it, and the exact replacement text — **do not** write a `## Conflict` block for it and do not put it to the owner. Sweep for *all* copies before reporting: a deleted rule usually left its restatement in more than one place, and reporting the first one found lets the others survive the cleanup.
   - **Live conflict** — two current statements and it is genuinely unclear which is in force, or the answer costs something either way. Only these get the block below and only these reach the owner.

   The test is not how important the contradiction is; it is whether a decision exists to be made. A stale restatement of an owner's own deleted rule is often the more embarrassing finding, and it is still not a question.

   For each **live conflict**, the report gives a block containing all of:
   - **A** — the rule verbatim, with source `file:line`, and its date or session if recoverable
   - **B** — the same, for the conflicting rule
   - **The collision** — the concrete situation in which following one violates the other
   - **In force today** — two determinations, which may disagree: which rule is *law* under the project's precedence order (owner ruling > `## Decisions` sections, root and directory-scoped > CLAUDE.md prose > MEMORY.md > docs/ > code comments > code), and which rule the code *actually obeys*, with evidence. When law and fact differ, say so plainly — that difference is the finding
   - **If A is revoked** — what depends on A: code, docs, workflows, other rules that cite it; what would have to change, and what breaks
   - **If B is revoked** — the same
   - **Recommendation** — keep A, keep B, or a new rule superseding both, with the reasoning. Recommend; the owner rules.
   - **Cleanup once ruled** — the exact files and lines to delete so the losing rule leaves no trace anywhere it is recorded. A list to be executed after the decision, not executed here. Cleanup never touches `completed_tasks.md` or any dated history log — a revoked decision was still truly made at its time, and the ruling session's own entry documents the supersession.
   - **Ready to file** — the same contradiction in the drift-review conflict format below, which you file at the end of the audit (see *Before the audit is done*).

     ```
     ## Conflict [n] — [short subject]

     - **Decided:** [rule A, by ID where one exists] ([source file:line]; Session [X], [date] if recoverable)
     - **Authority:** owner ruling | recorded decision | doc | code comment | code behaviour
     - **Now:** [rule B, or what the code actually does] ([source])
     - **Conflict:** [why the two cannot both stand]
     - **Raised:** Session [N], [date] by vitruvius audit
     - **Status:** awaiting owner ruling
     ```

     `Authority:` records what the `Decided:` side actually is. The rulings the owner will be offered — keep the earlier decision / adopt the new direction / not a conflict — are worded for something he decided; a block sourced from a stale code comment must not reach him dressed as his own past ruling. `Raised:` carries a session number, not just a date: the re-raise chain annotates it. If the session number is not recoverable from the project's records, say `Session unknown` rather than dropping the field.

     (This block format is owned by the drift-review skill, which consumes these blocks — if either copy changes, change both, and verify they match rather than trusting this note: they have drifted before while each claimed parity.)

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

## Before the audit is done — file the conflicts

**The audit is not finished when the report is written.** It is finished when every dimension-3 contradiction has a home that something will read again.

The report lives at `docs/reviews/vitruvius-YYYY-MM-DD.md`. Under the project's own file taxonomy that is a **record** — dated evidence, not auto-loaded, read only when someone goes looking. So a contradiction that stops there is a live, evidenced conflict in a file nothing opens: the SessionStart hook reads only `UNRESOLVED.md`; drift review reads only `.session_prompts/` and `MEMORY.md`; wrap-up's blocking check reads only `UNRESOLVED.md`. Six findings in a report, two hand-copied out, four gone — and nothing anywhere is wrong, so nothing complains. The **Ready to file** field makes promotion one paste, which is exactly what makes forgetting it a silent, one-keystroke loss.

Only **live conflicts** are filed. A stale restatement is not filed, not put to the owner, and not carried — it is a finding the project's session fixes under standing authority, and filing it as a conflict wastes a ruling on a question with one answer.

So, as the closing step, for **every** conflict block in the report:

1. `Read` `<repo_root>/.session_prompts/drift-reviews/UNRESOLVED.md` (`<repo_root>` = `Bash(git rev-parse --show-toplevel)`; create the file and its directories if absent — global CLAUDE.md, Precedence, says "create if absent", and a project without prompt capture still needs somewhere to hold the question).
2. Append each block that is not already there. Match on subject and sources, not on wording — re-filing a block the last audit already filed doubles the owner's queue and trains him to ignore it.
3. Number the blocks continuing from the file's existing highest `## Conflict [n]`, so `[n]` stays unique within the file.

Then tell the owner, in the run's closing message: how many conflicts the audit found, how many were newly filed, how many were already open, and — in one plain sentence each — what they are. Say plainly that they will be raised at the start of every session until he rules on them, and that `/drift-review` Phase 2 is where the rulings happen.

**You do not run the ruling loop.** No `AskUserQuestion`, no menu, no applying an outcome — an audit is long and it would land the interruption at the worst moment, and rulings are drift-review's job. Filing is the handoff; the hook does the rest. What you must never do is finish an audit reporting contradictions you left with no watcher.

Do not paste conflict blocks into `UNRESOLVED.md` still wrapped in the report's code fences: the hook counts lines beginning `## Conflict` without fence awareness, and a fenced or quoted example block wedges the blocking injection permanently on with nothing actually open.

The report's dimension-3 section states, per contradiction, that it was filed and where.
