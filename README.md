# amplifier-bundle-conformance

**The Amplifier-way auditor for bundle repos.**

`validate-bundle-repo` answers *"does it load?"* This bundle answers the second
question it can't: *"is it shaped right for the ecosystem?"* It scans an
`amplifier-bundle-*` repo against a codified rubric and returns **scored,
severity-tiered, file:line-cited findings** — then runs an audit → fix → re-audit
remediation loop. It's `impeccable`, but for **bundle architecture** instead of
visual design.

## Why this exists

Authoring a bundle has two halves. The first — does it parse, does `bundle.dot`
regenerate, do modules load — is covered by `validate-bundle-repo`. The second —
is it a *thin* bundle, are behaviors actually wired, are heavy docs context-sunk
into the agent that needs them, are `@mentions` used correctly, did you avoid
reinventing what foundation already gives you — has had **no repeatable, scored
check**. People do it by hand, inconsistently, every time they ship a bundle. This
bundle codifies that audit.

## What's in the box

| Piece | Path | Role |
|---|---|---|
| **Auditor agent** | `agents/auditor.md` | Reads the repo, applies the rubric, cites foundation docs, and escalates genuine judgment calls to the experts (see below). |
| **The rubric** | `context/rubric.md` | The codified, severity-tiered checks — each grounded in a foundation-doc §heading (referenced, not copied). |
| **`/audit-bundle` skill** | `skills/audit-bundle/` | The entry point: `/audit-bundle <repo-path>`. |
| **Remediation recipe** | `recipes/audit-and-fix.yaml` | Staged: audit → **your approval** → fix → re-audit. |
| **Awareness** | `context/conformance-awareness.md` | Thin always-on routing (<500 tokens). |

## Install

```bash
amplifier bundle add git+https://github.com/michaeljabbour/amplifier-bundle-conformance@main && amplifier update
```

Or add it as an **app bundle** by including it in your bundle's `includes:`:

```yaml
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: git+https://github.com/michaeljabbour/amplifier-bundle-conformance@main
```

## Use

```bash
# Skill entry point (after first push — see note below)
/audit-bundle ~/dev/amplifier-bundle-foo

# Auditor agent directly (works immediately, no push needed)
delegate(agent="conformance:auditor",
         instruction="Audit the amplifier bundle repo at ~/dev/amplifier-bundle-foo")

# Full remediation loop (audit → approve → fix → re-audit)
amplifier tool invoke recipes operation=execute \
  recipe_path=conformance:recipes/audit-and-fix.yaml \
  context='{"target_path": "~/dev/amplifier-bundle-foo"}'
```

> **Note on the `/audit-bundle` skill:** `tool-skills` discovers this bundle's
> `skills/` dir via a git self-reference (`#subdirectory=skills`) — the only source
> form that resolves independent of runtime cwd. It fetches from remote `HEAD`, so
> the skill becomes visible after the **first `git push` + `amplifier update`**.
> Until then, use the auditor agent directly (it needs no push). This is the same
> documented trade-off used by `amplifier-bundle-occams-machete`.

## Design notes

- **Thin bundle.** Inherits `amplifier-foundation` (filesystem, grep, bash, the
  foundation experts); adds exactly one capability via `behaviors/conformance.yaml`.
  No local tool module — the auditor uses inherited foundation tools.
- **The rubric references, it doesn't copy.** Each check cites a `BUNDLE_GUIDE.md`
  §heading so the standard ages with its source instead of drifting.
- **The auditor is a context sink.** The heavy rubric loads only when it runs; it
  defers genuine judgment calls to the experts rather than duplicating their
  knowledge — **structural/shape** questions to `foundation:foundation-expert`,
  **ecosystem-existence** questions ("does this already exist? should it be its own
  module?") to `amplifier:amplifier-expert`.
- **The recipe gates before editing.** No file changes until you approve the
  findings.

Verified by `amplifier:amplifier-expert` and `foundation:foundation-expert`
(both GO) before it was built.
