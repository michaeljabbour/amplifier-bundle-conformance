---
name: audit-bundle
description: "Audit an amplifier-bundle-* repo for Amplifier-way conformance — not 'does it load' (that's validate-bundle-repo) but 'is it shaped right for the ecosystem.' Returns scored, severity-tiered, file:line-cited findings grounded in the foundation docs. Use before a bundle PR, after a structural change, or when a bundle 'feels off' and you want to know why."
context: fork
disable-model-invocation: true
user-invocable: true
model_role: critique
---

# /audit-bundle — Amplifier-Way Conformance Audit

Audit a bundle repository against the Amplifier-way rubric and return scored findings.

## Target

`$ARGUMENTS` is the path to the bundle repo to audit. If empty, audit the current
working directory. Resolve it to an absolute path and confirm a root `bundle.md`
exists there — if it doesn't, say so and stop (this skill audits bundle repos).

## What to do

1. **Delegate the audit to the auditor agent** — it carries the rubric and the
   method as a context sink:

   ```
   delegate(
     agent="conformance:auditor",
     instruction="Audit the amplifier bundle repo at <resolved-path>. Apply the full
                  conformance rubric. Read the repo's own bundle.md body and docs/ for
                  stated intent before flagging. Return the scored report in your
                  standard format, with a file:line and a foundation-doc citation for
                  every finding.",
     context_depth="none"
   )
   ```

2. **Present the auditor's report verbatim** — the score, the verdict, every finding
   with its `file:line` and authority citation, what the bundle does right, and any
   open questions it escalated.

3. **Offer the remediation loop.** End with one line:
   > Want these fixed? Run `conformance:recipes/audit-and-fix.yaml` (context
   > `{"target_path": "<resolved-path>"}`) — it re-audits, pauses for your approval,
   > applies fixes, and re-audits to confirm.

Do not invent findings beyond the auditor's report, and do not soften ERRORs — relay
them as the auditor scored them.
