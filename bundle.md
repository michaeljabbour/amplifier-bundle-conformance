---
bundle:
  name: conformance
  version: 0.2.1
  description: >-
    An Amplifier-way conformance auditor for bundle repos. Where validate-bundle-repo
    asks "does it load?", this asks "is it shaped right for the ecosystem?" — it scans
    an amplifier-bundle-* repo against a codified rubric (ruthless simplicity,
    bricks-and-studs, mechanism-not-policy, the thin-bundle pattern, context-sink
    agents, correct @mention usage, not reinventing foundation) and emits scored,
    file:line-cited findings, then runs an audit → fix → re-audit remediation loop.
    Ships an auditor agent, the rubric, an /audit-bundle skill, and an audit-and-fix
    recipe.

# Thin bundle: inherit foundation (filesystem, grep, bash, web, the foundation
# experts) and then add the one unique capability via the behavior. The behavior
# wires the auditor agent, the /audit-bundle skill, and the audit-and-fix recipe.
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: conformance:behaviors/conformance
---

# Conformance — the Amplifier-Way Auditor

`validate-bundle-repo` answers one question: **does it load?** That is necessary
but not sufficient. A bundle can parse cleanly, regenerate its `bundle.dot`, and
still be *shaped wrong for the ecosystem* — a fat bundle that redeclares what
foundation already gives you, a behavior file sitting on disk that nothing
includes (silently inert), heavy docs pinned always-on instead of living in the
agent that needs them, `@`-prefixed mentions in YAML that quietly fail.

This bundle answers the **second** question: **is it shaped the amplifier way?**

It does for *bundle architecture* what `impeccable` does for *visual design* —
scans a repo against a codified rubric and returns scored findings you can act on,
each with a file:line and a citation back to the authoritative foundation doc.

## What's in the box

| Capability | What it is | Reach for it when |
|---|---|---|
| `/audit-bundle <repo-path>` | The entry point. Audits a bundle repo and returns scored, severity-tiered findings. | You're about to open a bundle PR, or just finished a structural change. |
| `delegate(agent="conformance:auditor")` | The auditor — reads the repo, applies the rubric, cites foundation docs, defers ecosystem judgment to `foundation:foundation-expert`. | You want the findings directly, mid-task. |
| `conformance:recipes/audit-and-fix.yaml` | The remediation loop: audit → **you approve** → fix → re-audit. | You want the issues *fixed*, not just listed. |
| `conformance:context/rubric.md` | The codified rubric — the checkable rules, grounded in `BUNDLE_GUIDE.md`. | You want to read (or argue with) the standard itself. |

The rubric **references** the foundation docs rather than copying them, so it
ages with the source instead of drifting from it. The auditor is a context sink:
it loads the heavy rubric only when it runs, and it leans on the existing
`foundation:foundation-expert` for judgment calls rather than re-inventing what
foundation already knows.

Start at `README.md`.
