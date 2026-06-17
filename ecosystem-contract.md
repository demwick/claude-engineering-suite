<!--
  claude-engineering-suite
  Ecosystem contract — the single canonical source for the boundaries and
  version compatibility of the three composing projects. Each project's
  README should LINK here rather than restate these rules, so the boundary
  definition lives in exactly one place and cannot drift between repos.
-->

# Ecosystem Contract

Three independent projects compose into one human-in-the-loop AI engineering
workflow. They share no code. Each works fully standalone. When installed
together they **hand off** responsibilities at well-defined seams rather than
duplicating them.

This file is the **one canonical definition** of those seams. If a project's
own docs disagree with this file, this file wins — fix the project's docs.

> **Versions described here:** `software-engineer` 4.3.0 · `claude-charter`
> v0.1.3 · `centaur-layer` 0.2.0. See the [compatibility matrix](#version-compatibility)
> for the tested combinations.

## The three layers

Each layer acts at a **different moment**, on a **different surface**, and
watches a **different actor**. That orthogonality is the whole design — it is
why they do not collide and why none should be merged into another.

| Project | Role | Moment | Surface | Watches | Format | License |
| --- | --- | --- | --- | --- | --- | --- |
| **software-engineer** | the **engine** — triage → clarify → spec → ADR → risk → plan → execute → verify | planning & build time (before a change is written) | subagents + `Stop`/`SessionStart`/`PostToolUse` hooks | the work | Claude Code plugin | AGPL-3.0-or-later |
| **claude-charter** | the **constitution** — policy, trust boundaries, guardrails, ADR/verify *format* | continuously + `PreToolUse` (before the AI invokes a tool) | layered policy (`charter/`→`context/`→`evidence/`) + `PreToolUse` hook | the **machine** | `./install` template (not a plugin) | MIT |
| **centaur-layer** | the **brake** — diff-risk score + human comprehension check | acceptance time (`pre-commit`, before the human accepts) | git `pre-commit` hook + GitHub Action | the **human** | dual plugin (Claude Code + Codex) | MIT |

**One-line mnemonic:** charter says *"AI, you may not do X"*; the engine says
*"this change is risky, watch for X"*; centaur says *"human, do you understand
why X changed?"* — three sentences, three speakers, three moments.

## Ownership — who owns what

A responsibility belongs to exactly one layer. The others defer to the owner
when it is present.

| Responsibility | Owner | Notes |
| --- | --- | --- |
| Layered policy & trust boundaries | charter | `charter/` (policy) vs `context/` (trusted facts) vs `evidence/` (untrusted) |
| `PreToolUse` destructive-op guardrails | charter | asks before `rm -rf`, `git push --force`, `.env` reads, etc. — never silently blocks |
| ADR **format** & canonical location | charter | `.claude/knowledge/adr/0000-template.md` |
| `/verify` **definition** + verdict format | charter | adversarial `PASS \| FAIL \| PARTIAL` |
| Triage / clarify / spec / plan / execute | engine | the actual engineering pipeline |
| Forward-looking **risk warning** | engine | warns at plan time, before the change exists |
| ADR **generation** | engine | produces the record, writing it in charter's format & location when charter is present |
| `/verify` **execution** | engine | the `verifier` subagent runs it, inheriting charter's role |
| Acceptance-time **diff-risk scoring** | centaur | deterministic low/med/high at `pre-commit` |
| Human **comprehension gate** | centaur | questions the developer, never the machine |

## Detect & Defer — the handoff rules

Detection is by **directory probe**, automatic, read-only. No project writes to
another's state directory.

- **engine detects charter** (`.claude/knowledge/charter/` exists) → write ADRs
  to charter's `.claude/knowledge/adr/` in charter's template (not `.se/adr/`);
  add **no** `PreToolUse` guardrail (charter owns that); emit charter's
  `PASS/FAIL/PARTIAL` verdict instead of its own severity format.
- **engine detects centaur** (`.se/.centaur` or `.claude/knowledge/centaur/`) →
  keep risk strictly forward-looking; do **not** score the committed diff
  (centaur owns acceptance-time scoring).
- **charter detects engine** (`.se/` exists) → defer `/verify` execution and
  automatic ADR generation to the engine; keep owning the format, location, and
  the 12-point `/health` self-audit.
- **centaur detects charter and/or engine** → respect both, reimplement
  neither; keep its own `pre-commit` hook (a different surface, not a repeat of
  charter's `PreToolUse` gate).

Standalone, each layer does its own job end to end. The result is the
[no-collision invariants](#no-collision-invariants) below.

## No-collision invariants

These must hold for any combination of installed layers. A violation is a bug
in whichever layer broke it.

1. **One event, one guard.** charter guards `PreToolUse`; centaur guards
   `pre-commit`; the engine's `Stop` hook guards post-change test runs. No two
   layers gate the same event.
2. **One owner per responsibility.** The ownership table above is exhaustive.
   If two layers both act on a responsibility, one is deferring incorrectly.
3. **No cross-writes.** No layer writes to another's state directory
   (`.se/`, `.claude/knowledge/`, `.centaur/`). Cross-project reads are
   read-only detection probes only.
4. **Standalone integrity.** Removing any layer leaves the others fully
   functional; none hard-requires another.

## Version compatibility

Tested-compatible combinations. "Compatible" means the Detect & Defer seams and
the no-collision invariants hold for that triple.

| software-engineer | claude-charter | centaur-layer | Status | Notes |
| --- | --- | --- | --- | --- |
| 4.3.0 | v0.1.3 | 0.2.0 | ✅ current baseline | all three probe-detect each other; seams verified by each project's own docs |
| 4.2.0 | v0.1.3 | 0.2.0 | ✅ compatible | 4.3.0 added gate taxonomy + must-haves — internal, no seam change |
| 4.x | v0.1.x | 0.2.x | ✅ expected | same Detect & Defer contract; minor bumps are additive |
| ≤ 3.x | — | — | ⚠️ pre-triage | engine ≤ 3.x used `/se-init`,`/se-go`,`/se-quick`; charter's engine-deferral assumes the v4 triage surface |

**Rule of thumb:** the seams are defined by *directory contracts* (the probed
paths above), not by versions. Any versions that keep those paths and the
ownership table stay compatible. When a layer changes a probed path or moves a
responsibility, it MUST update this file in the same change.

## Format asymmetry (known, deliberate)

Two of the three are Claude Code plugins; charter is a `./install` template.
This is intentional: charter distributes **policy** (versioned, diffable,
layered for prompt-injection defense), which is a different artifact from
executable plugin logic. Consequence: a Claude Code marketplace can bundle the
engine and centaur as plugins, but **charter is installed separately** via its
`./install <target>` command. The suite README documents both paths.

## License note

`software-engineer` is **AGPL-3.0-or-later**; `claude-charter` and
`centaur-layer` are **MIT**. The suite as a whole is a *distribution
convenience*, not a combined work — each project keeps its own license. If you
redistribute a modified `software-engineer` as a hosted service, AGPL's
network-use clause applies to that component only.

## How to reference this file

Each project's README should carry a single line under its "ecosystem" section:

> Boundaries and version compatibility are defined canonically in
> [`ecosystem-contract.md`](https://github.com/demwick/claude-engineering-suite/blob/main/ecosystem-contract.md).
> This README summarizes; the contract governs.

That keeps the boundary definition in one place and turns three drifting
restatements into three pointers.
