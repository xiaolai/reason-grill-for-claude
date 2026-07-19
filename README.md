# reason-grill

Deep interrogation of an **argument** — the way [grill](https://github.com/xiaolai/grill-for-claude) interrogates a codebase. One command maps a claim, attacks it from five adversarial angles, and returns a calibrated verdict on whether it survives.

## What it does

grill finds bugs in code. reason-grill finds flaws in *reasoning* — in an essay, a proposal, a PR description, a design rationale, or any claim-plus-argument. It maps the argument, then runs five specialist agents that each attack a different angle, and reduces their findings to one verdict.

The hard part — and the reason most "AI critique" is worthless — is that **an argument has no compiler.** Code compiles or it doesn't; an argument has no automatic backstop, so a critic's default failure mode is manufacturing plausible-sounding objections. reason-grill's answer is a **six-check calibration gate** that every objection must survive, plus one structural trick: every finding must print *the author's strongest one-line rebuttal to itself*. An objection that dies to its own rebuttal never reaches you.

The practical consequence: **on a sound argument, most angles return `[SOUND]`.** A clean bill is the expected result, not a cop-out.

- **6 agents**: `thesis` (maps the claim), then `logic`, `evidence`, `counter`, `assumptions`, `integrity`
- **5 review styles**: Full grill, Quick pressure test, Steelman-first, Devil's-advocate panel, Pre-mortem
- **Calibrated verdict**: `HOLDS` / `HOLDS-IF-NARROWED` / `WOUNDED` / `DEFEATED`, with the single strongest surviving objection and a confidence level

## Installation

### Claude Code

Via the xiaolai marketplace:

```
/plugin marketplace add xiaolai/claude-plugin-marketplace
/plugin install reason-grill@xiaolai
```

> **Install fails with "Plugin not found in marketplace 'xiaolai'"?** Your local marketplace clone is stale. Run `claude plugin marketplace update xiaolai` and retry — `plugin install` does not auto-refresh.

| Scope | Command | Effect |
|-------|---------|--------|
| **User** (default) | `/plugin install reason-grill@xiaolai` | Available in all your projects |
| **Project** | `/plugin install reason-grill@xiaolai --scope project` | Shared with team via `.claude/settings.json` |
| **Local** | `/plugin install reason-grill@xiaolai --scope local` | Only you, only this repo |

## Usage

```
/reason-grill:roast
```

The argument can be inline text, a file path, or a URL:

```
/reason-grill:roast docs/proposal.md
/reason-grill:roast https://example.com/blog/why-we-should-rewrite-in-rust
```

## How it works

1. **Map** — the `thesis` agent extracts the central claim, its premises (stated and implicit), the inference structure, and the evidence offered. No critique — just the target.
2. **Choose a style** — Full grill (all five angles), or a focused variant.
3. **Deep dive** — the attacker agents run in parallel, each applying the calibration gate.
4. **Synthesize** — findings are deduplicated, the gate is re-run one last time (any objection defeated by its own printed rebuttal is dropped), and what remains is ranked by damage to the conclusion.
5. **Verdict** — one line on whether the argument holds, the strongest surviving objection, and how confident that verdict is.

## Analysis angles

| Angle | Grants | Attacks |
|-------|--------|---------|
| `thesis` | — | *(maps the argument; does not critique)* |
| `logic` | premises are true | does the conclusion *follow*? (non-sequitur, equivocation, circularity, quantifier slip) |
| `evidence` | the inference is valid | are the premises *true and calibrated*? (unsupported claims, over-confidence, cherry-picking) |
| `counter` | — | the strongest steelmanned case *against* — does the argument survive it? |
| `assumptions` | — | hidden load-bearing premises and boundaries where the claim breaks |
| `integrity` | — | is the *framing honest*? (motte-and-bailey, dodged objections, loaded words) |

All five attackers share the `reason-grill-core` skill, which defines the calibration gate, severity scale, and finding format.

## Severity tags

Severity measures damage to the **conclusion**, not taste in how it's argued:

- `[FATAL]` — Defeats it. If correct, the thesis is false or unsupported.
- `[MAJOR]` — Wounds it. Needs real repair to survive.
- `[MINOR]` — Weakens but doesn't defeat. Patchable.
- `[POLISH]` — Robustness or clarity note; the conclusion holds regardless.
- `[SOUND]` — Nothing passed the gate on this angle. The argument holds here.

## Relationship to other plugins

- **grill** — same architecture, applied to code.
- **claude-english-buddy** — reviews *how* you write (grammar, tone, clarity). reason-grill reviews *whether the reasoning is valid*. english-buddy passes a beautifully written fallacy; reason-grill catches it.

## License

ISC
