# Agent Instructions — amplifier-bundle-conformance

This repo is an Amplifier bundle: the **Amplifier-way conformance auditor**. It
scores a bundle repo against a codified rubric and emits file:line-cited findings.

## Layout

| Path | What it is |
|---|---|
| `bundle.md` | Thin root: includes foundation + `behaviors/conformance`. |
| `behaviors/conformance.yaml` | Wires the auditor agent, `/audit-bundle` skill (tool-skills), and recipe (tool-recipes). |
| `agents/auditor.md` | The auditor — context sink; @mentions the rubric only. |
| `context/rubric.md` | The standard. Each check cites a foundation-doc §heading (referenced, not copied). |
| `context/conformance-awareness.md` | Thin always-on routing (<500 tokens). |
| `skills/audit-bundle/SKILL.md` | `/audit-bundle <repo-path>` entry point (fork skill). |
| `recipes/audit-and-fix.yaml` | Staged: audit → approval → fix → re-audit. |

## Conventions / pitfalls

- **Keep `bundle.md` thin.** Do not redeclare foundation tools, `session:`, or core
  hooks. New capability goes in a behavior, not inline. (This bundle audits for that
  — it must pass its own rubric.)
- **The rubric references, never copies.** When a check needs detail, cite the
  `BUNDLE_GUIDE.md` §heading; don't paste the rule. This keeps it from drifting.
- **The `/audit-bundle` skill needs a push.** `tool-skills` discovers `skills/` via a
  git self-reference fetched from remote `HEAD`. After editing `skills/`, push +
  `amplifier update` before it's visible. The auditor agent works without a push.
- **Dogfood it.** Run `delegate(agent="conformance:auditor", ...)` against this very
  repo (and other bundles) after any change — it should pass its own audit.

## Test / verify

There is no Python module here, so no unit suite. Verification is behavioral:
1. `delegate(agent="conformance:auditor", instruction="Audit the bundle repo at .")`
   against this repo and a known-good + known-bad bundle.
2. Confirm every finding has a `file:line` and a foundation-doc citation.
3. Confirm legitimate exceptions (stated in a bundle's `docs/`) are NOT flagged.
