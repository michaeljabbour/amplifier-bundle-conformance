# Amplifier-Way Conformance (awareness)

This session can audit a bundle repo for **amplifier-way conformance** — not "does
it load" (that's `validate-bundle-repo`) but "is it *shaped right* for the
ecosystem." It's `impeccable`, but for bundle architecture instead of visual design.

## Ways in

- **`/audit` mode** — a sustained *audit posture*: treat the working repo (or a
  named path) as a bundle to audit; findings are scored + file:line-cited; direct
  edits are warned so fixes route through the gated recipe. `/mode off` to leave.
- **`/audit-bundle <repo-path>`** — one-shot entry point. Scores a bundle repo and
  returns severity-tiered, file:line-cited findings. (Until this bundle's first
  `git push`, use the agent directly — see below.)
- **`delegate(agent="conformance:auditor", instruction="Audit the bundle at <path>")`**
  — the auditor directly, mid-task.
- **`conformance:recipes/audit-and-fix.yaml`** — the remediation loop:
  audit → **you approve** → fix → re-audit.

## When to reach for it

Before opening a bundle PR; after a structural change to a bundle; or when a
bundle "feels off" and you want to know *why*, with citations.

The standard itself lives in `conformance:context/rubric.md` — it **references**
the foundation docs rather than copying them, and the auditor defers genuine
ecosystem judgment to `foundation:foundation-expert` rather than guessing.
