# Vitruvius

*Firmitas. Utilitas. Venustas.*

A [Claude Code](https://claude.com/claude-code) skill that audits a codebase the way a Roman engineer judged a building — **does it stand, does it serve, is anything superfluous** — and hands you one honest, evidence-anchored report.

It never touches your code. That is the point.

## Why "Vitruvius"

Marcus Vitruvius Pollio, engineer and architect of the 1st century BC, wrote *De architectura* for the Emperor Augustus: ten books defining what makes a building good, condensed into three tests that survived twenty centuries — **firmitas** (soundness), **utilitas** (fitness for purpose), **venustas** (elegance). Every structure he worked on is dust. The written assessment outlived them all.

This skill is built on the same premise: an honest written audit outlives any individual fix — and any individual model. Its dimensions are Vitruvius' tests, translated:

| Vitruvius asked | The skill audits |
|---|---|
| **Firmitas** — will it stand? | Foundations & correctness — architecture, error handling, bugs, security; what works today but breaks when something shifts |
| **Utilitas** — will it serve as demands grow? | Scalability & future-proofness — concurrency, resource growth, single-user assumptions |
| **Venustas** — is anything superfluous? | Leanness — dead code, duplication, contradictions, glue and scar tissue from repeated AI-assisted rewrites; and above the single file, the shape of the whole: how the modules cooperate, whether a simpler route would do the same job with fewer failure points, and an explicit verdict — elegant or spaghetti — with the prescribed untangling |

Plus a fourth dimension the treatise-writer would have recognized: **the free take** — the auditor's unconstrained view of better approaches and what to build next, once it truly knows your codebase.

## What it does

One command from a project root:

```
/vitruvius
```

One deliverable: `docs/reviews/vitruvius-YYYY-MM-DD.md` in the audited project.

- **It reads the story before the code.** Project docs, decision logs, git history — because the most valuable findings are often invisible in code alone: *zombie decisions*, code still faithfully implementing what was cancelled weeks ago. If you work with AI agents daily, you know this species of debt.
- **Every claim is anchored.** `file:line`, a commit, a config entry, or observed command output. What breaks, and under exactly what conditions. Anything unprovable is labeled UNVERIFIED instead of asserted.
- **Findings are ranked by effort-versus-benefit** — quick wins first, each with a severity and a time estimate — and each dimension closes by naming the single change that would most improve it.
- **It ends with a blunt bottom line.** The codebase's honest condition in a few sentences, written in plain language a non-programmer owner can act on.

## What it will never do

Edit anything. Not a typo, not an unused import, not a "safe" cleanup. An obvious, cheap fix is a finding, not an action — the report file is the only thing it writes.

This is a hard rule because it fails without one: offer an agent a friendly *"handle small problems as you see fit"* and it will start fixing mid-audit. An audit that edits is neither an audit nor safe.

## How it was tested

The skill was developed test-first, the way you'd develop code:

1. **Baseline (red):** a fresh agent was given a real codebase and a naive audit request, including the tempting phrase *"handle small obvious problems as you see fit."* It edited three files mid-audit and delivered its findings as chat — which evaporates.
2. **Skill (green):** written specifically against those observed failures.
3. **Re-test:** same codebase, same temptation, verbatim. Zero files touched, and a ranked, anchored, four-dimension report saved to disk — with the temptation itself politely declined in writing.

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

The skill is deliberately lean — 49 lines. It fixes only what must not drift (the no-edit rule, evidence anchoring, the four dimensions, the report contract) and leaves method, judgment, and delegation to the model. It was authored and validated during the Claude Fable 5 window, following Anthropic's guidance that over-prescriptive skills degrade strong models. It runs on whatever your session uses, but it checks the engine first: below a Fable-class model at high reasoning effort it asks the operator before proceeding, and every report opens with a provenance line naming the model and effort that produced it.

## License

MIT — see [LICENSE](LICENSE).

Built by [Cemil Gunes](https://github.com/djemil), Fabervant Teknolojileri, with Claude.
