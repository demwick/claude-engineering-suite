# Claude Engineering Suite

> **One human-in-the-loop AI software-engineering workflow, composed from three
> independent layers — each acting at a different moment, on a different
> surface, watching a different actor.**

This repo is a **thin façade**, not a fourth product. It carries no engineering
code. It exists to solve the one weakness of a three-project design: the
adoption funnel and the boundary definition were fragmented across three
READMEs. Here they are unified — one install path, one narrative, one canonical
[`ecosystem-contract.md`](./ecosystem-contract.md).

The three engines stay in their own repos, with their own licenses and release
cycles. Merging their code would tangle three orthogonal surfaces and three
versioning lifecycles — see the contract for why. This suite unifies *how you
adopt and reason about them*, not *how they are built*.

## The three layers

| Layer | Role | Acts at | Asks |
| --- | --- | --- | --- |
| [**software-engineer**](https://github.com/demwick/software-engineer) | the **engine** | planning & build time | *"here's how this work should go — and what it might break"* |
| [**claude-charter**](https://github.com/demwick/claude-charter) | the **constitution** | `PreToolUse` (before the AI acts) | *"AI, you may not do X"* |
| [**centaur-layer**](https://github.com/demwick/centaur-layer) | the **brake** | `pre-commit` (before you accept) | *"human, do you understand why X changed?"* |

Different moments, no collision. The full boundary definition and the version
compatibility matrix live in **[`ecosystem-contract.md`](./ecosystem-contract.md)**
— the single source of truth all three projects point to.

## Install

Two of the three are Claude Code plugins and install from this suite's
marketplace. The third (charter) is a policy **template** and installs into your
project separately. (This asymmetry is deliberate — see the contract's
[format note](./ecosystem-contract.md#format-asymmetry-known-deliberate).)

### 1. The plugins — engine + brake

```bash
# Add this marketplace, then install either or both plugins
/plugin marketplace add demwick/claude-engineering-suite
/plugin install software-engineer@claude-engineering-suite
/plugin install centaur-layer@claude-engineering-suite
```

### 2. The constitution — charter (separate, per-project)

```bash
git clone https://github.com/demwick/claude-charter
cd claude-charter
./install /path/to/your/project
```

## What to install for which need

You do **not** need all three. Each works standalone; they compose when present.

| You want… | Install |
| --- | --- |
| AI that does the engineering (spec, plan, implement, test, review) | **software-engineer** |
| A pre-commit brake that keeps *you* understanding the diff | **centaur-layer** |
| A versioned policy + guardrail + trust-boundary layer | **claude-charter** |
| The full human-in-the-loop workflow | all three — they auto-detect and hand off |

When more than one is present, each detects the others (read-only directory
probes) and **defers** the responsibilities the others own — no duplication, no
configuration. That handoff is specified in the contract's
[Detect & Defer rules](./ecosystem-contract.md#detect--defer--the-handoff-rules).

## Versions

This suite pins a **known-good baseline**: `software-engineer` 4.2.0 ·
`claude-charter` v0.1.3 · `centaur-layer` 0.2.0. To ship a newer compatible
triple, bump the versions in [`marketplace.json`](./.claude-plugin/marketplace.json)
and the matrix in the contract together — they must move as one.

## Licenses

`software-engineer` is **AGPL-3.0-or-later**; `claude-charter` and
`centaur-layer` are **MIT**. This suite is a distribution convenience, not a
combined work — each project keeps its own license.
