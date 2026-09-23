# Vitruvius

*Firmitas. Utilitas. Venustas.*

A [Claude Code](https://claude.com/claude-code) skill that audits a codebase the way a Roman engineer judged a building — **does it stand, does it serve, is anything superfluous** — and hands you one honest, evidence-anchored report that ends by naming **one** part of the codebase the project must refine next.

It never touches your code. That is the point.

## Why "Vitruvius"

Marcus Vitruvius Pollio, engineer and architect of the 1st century BC, wrote *De architectura* for the Emperor Augustus: ten books defining what makes a building good, condensed into three tests that survived twenty centuries — **firmitas** (soundness), **utilitas** (fitness for purpose), **venustas** (elegance). Every structure he worked on is dust. The written assessment outlived them all.

This skill is built on the same premise: an honest written audit outlives any individual fix — and any individual model. Its dimensions are Vitruvius' tests, translated:

| Vitruvius asked | The skill audits |
|---|---|
| **Firmitas** — will it stand? | Foundations & correctness — architecture, error handling, bugs, security; what works today but breaks when something shifts |
| **Utilitas** — will it serve as demands grow? | Scalability & future-proofness — concurrency, resource growth, single-user assumptions |
| **Venustas** — is anything superfluous? | Leanness & simplification — dead code, redundant branches, indirection that isolates nothing, glue and scar tissue from repeated AI-assisted rewrites; and above the single file, the shape of the whole: how the modules cooperate, whether a simpler route would do the same job with fewer failure points, and an explicit verdict — elegant or spaghetti — with the prescribed untangling. The required shape is a tree: small units used by bigger ones, every task built once, low down, and reached from above; the verdict is *a tree*, *a tree with named broken branches*, or *no tree* |
| **Venustas**, again — is anything built twice? | Consistency & single source of truth — one thing implemented in several places: a button styled inline on two screens, two functions parsing the same date, the same rule validated twice. Every site named, the shared unit that should replace them, and the drift the copies have already produced. A required census in two tables (screen objects, and tasks the app performs) proves the sweep was whole; each family of variants (spacings, colours, error types, helpers) is held to a documented set, and a small one. Tests are in scope on the same terms: two tests asserting the same requirement are one test and a maintenance cost |
| **Firmitas**, in the documents | Doctrine contradictions — rules the project set itself that now contradict each other, or that the code no longer obeys. Both rules quoted with sources, the consequence of revoking each, a recommendation, and the exact cleanup list to run once you rule |

Plus a dimension the treatise-writer would have recognized: **the free take** — the auditor's unconstrained view of better approaches and what to build next, once it truly knows your codebase.

**The simplification standard.** `x + 4x² + x³ + 2x - x² + 1` and `(x+1)³` are the same function; one of them is a mess. Vitruvius reports the second form. Every such finding carries an *equivalence argument* — the current form, the proposed form, and why they behave identically for every input the current one handles, with the edge cases checked named. Simpler but not equivalent is a behavior change, and gets labeled one.

## The refactor mandate: the one output that obliges somebody

Reports alone did not work. Nine audits of one project left 35 findings still present and 7 of them grown: a finding had no owner and no exit condition, so the next session started on new content and the finding came back larger. So Vitruvius now feeds a refactoring program that obliges the project:

- **Each project refines its ten longest files, each in three separate sessions,** one round per session. The list rolls: whatever the ten longest are at each session start.
- **In each round a fresh reviewer** - an agent that did not do the work, reading a brief the author's hook prints so the worker cannot soften it - lists every remaining defect in the file: duplication, dead code, a patch on a patch, a knot, a split that should not exist, an unpinned or repeated test, a stale comment. Each simplification carries its equivalence argument. The session removes the defects. It may object to one, but an objection settles only by agreement: the reviewer answers, then a second fresh reviewer who reads the whole discussion, then the project's owner - never the worker's own word.
- **A round closes on a check, never on a declaration:** the list worked off, the cleanliness checks clean, and the code (code lines only - no comments, blanks, imports or tests) not grown. Moving code into new files is not refining. Perfectly refined code closes with nothing removed, because there is nothing left to remove; a fixed percentage would never let it stop.
- **Contract tests first,** written against the requirement, never against what the code does today. They never count against the round.

**Vitruvius's role is the surveyor; the round reviewer is the inspector.** The reviewer sees one file. Vitruvius sees the whole tree, so it names the target the program cannot see for itself - above all a scatter of small, shallow files that should be folded into one, which never ranks among the ten longest - or confirms the next file owed a round. It opens its report with the program's status: the ten longest files, reviews done of three, the open round, and its earlier findings still open. **Its CRITICAL and WARNING findings oblige too:** each is recorded, shown at every session start, and closed only by a fix or by the same agreement - a finding is never closed by relabelling the code "dormant" or "documented". Neither Vitruvius nor the reviewer edits code; the project's own session does the refining.

## What it does

One command from a project root:

```
/vitruvius
```

One deliverable: `docs/reviews/vitruvius-YYYY-MM-DD-S<session>.md` in the audited project. It opens with the refactoring program's status and the file named next, then one section per dimension. The session number in the filename matters: the author's scheduled-audit trigger reads it to decide when the next audit is due.

- **It reads the story before the code.** Project docs, decision logs, git history — because the most valuable findings are often invisible in code alone: *zombie decisions*, code still faithfully implementing what was cancelled weeks ago. If you work with AI agents daily, you know this species of debt.
- **Every claim is anchored.** `file:line`, a commit, a config entry, or observed command output. What breaks, and under exactly what conditions. Anything unprovable is labeled UNVERIFIED instead of asserted.
- **Findings are ranked by effort-versus-benefit** — quick wins first, each with a severity and what breaks, and when — and each dimension closes by naming the single change that would most improve it.
- **It ends with a blunt bottom line.** The codebase's honest condition in a few sentences, written in plain language a non-programmer owner can act on.

## What it will never do

Edit anything. Not a typo, not an unused import, not a "safe" cleanup. An obvious, cheap fix is a finding, not an action.

This is a hard rule because it fails without one: offer an agent a friendly *"handle small problems as you see fit"* and it will start fixing mid-audit. An audit that edits is neither an audit nor safe.

It writes exactly two things: the report, and the name of the next refactor round. Neither changes a line of your code.

## How it was tested

The skill was developed test-first, the way you'd develop code:

1. **Baseline (red):** a fresh agent was given a real codebase and a naive audit request, including the tempting phrase *"handle small obvious problems as you see fit."* It edited three files mid-audit and delivered its findings as chat — which evaporates.
2. **Skill (green):** written specifically against those observed failures.
3. **Re-test:** same codebase, same temptation, verbatim. Zero files touched, and a ranked, anchored, every-dimension report saved to disk — with the temptation itself politely declined in writing.

## Install

Copy the `vitruvius/` folder into your Claude Code skills directory:

```bash
git clone https://github.com/djemil/Vitruvius.git
cp -r Vitruvius/vitruvius ~/.claude/skills/
```

Windows (PowerShell):

```powershell
git clone https://github.com/djemil/Vitruvius.git
Copy-Item -Recurse Vitruvius\vitruvius "$env:USERPROFILE\.claude\skills\"
```

Then open Claude Code in any project root and run `/vitruvius`.

**One piece is not in this repository.** The refactoring program runs in `~/.claude/hooks/refactor_mandate.py` (the ranking, the reviewer's brief, the round records) with the author's session-start, edit and cleanliness gates enforcing it. Those hooks are not published. Without them the six dimensions are unaffected, but the mandate step has no program to read, no record to write, and nothing to enforce it.

The skill is deliberately lean — 140 lines. It fixes only what must not drift (the no-edit rule, evidence anchoring, the six dimensions, the content each finding must carry, the surveyor's part in the refactoring program, the report contract) and leaves method, judgment, and delegation to the model, following Anthropic's guidance that over-prescriptive skills degrade strong models. It runs on whatever your session uses, but it checks the engine first: below an Opus-5-class model at high reasoning effort it says so once and runs the audit anyway, and every report opens with a provenance line naming the model and effort that produced it.

## License

MIT — see [LICENSE](LICENSE).

Built by [Cemil Gunes](https://github.com/djemil), Fabervant, with Claude.
