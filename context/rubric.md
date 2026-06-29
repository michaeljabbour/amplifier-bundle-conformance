# The Amplifier-Way Conformance Rubric

<!--
CHANGELOG
  v0.2.1 — Dual-expert escalation: structural/shape open questions route to
    foundation:foundation-expert (default); ecosystem-existence questions
    ("does this already exist? should it be its own module?", the W1 axis) route
    to amplifier:amplifier-expert, which owns the MODULES.md catalog.
  v0.2.0 — Evidence-driven from real fleet audits (idd, letsgo, occams-machete):
    - NEW W7 "Orphaned context file" — a context/*.md referenced by nothing.
      Recurred 3× in a single bundle (letsgo); validated NON-false-positive on
      occams-machete (a context-sink @mention in an agent body counts as WIRED).
    - E4 — added the capability-only carve-out: a hook whose mount() does real
      work via coordinator.register_capability(...) is CLEAN, not a no-op stub.
    - E2 — scoped to bundle-LOADER YAML fields; excludes @ inside a module's own
      config: block (resolved by that module's mention logic, not the loader).
  v0.1.0 — Initial rubric (E1–E6, W1–W6, I1–I4).
-->

The standard the `conformance:auditor` applies. Every check below is grounded in an
authoritative foundation doc — cited by **section heading**, not paraphrased — so
that when the source doc changes, the rubric can be re-synced by searching for the
citation rather than silently drifting.

> **How to use this rubric.** For each check: gather concrete evidence (file +
> line + the offending token), read the bundle's own `docs/`/`bundle.md` body for a
> *stated* exception first, and only then record a finding. A check you cannot
> adjudicate from the repo alone is an **Open question** — delegate it to
> `foundation:foundation-expert`, do not guess. Hard violations and
> defaults-with-exceptions are different things; say which one you found.

Authority docs (available via the foundation bundle):
`foundation:docs/BUNDLE_GUIDE.md`, `foundation:docs/PER_REPO_CONVENTIONS.md`,
`foundation:docs/CONCEPTS.md`, `foundation:docs/URI_FORMATS.md`.

---

## ERROR — provably wrong or silently broken

You can point at the proof. These either break at load/run or are inert.

### E1 — Behavior on disk but never included (inert capability)
- **Detect:** a `behaviors/*.yaml` file exists, but the root `bundle.md` `includes:`
  list (transitively) never references it. It contributes nothing at runtime.
- **Authority:** BUNDLE_GUIDE.md § *Recommended Pattern* / § *Behavior Pattern*.
- **Note:** the inverse — `includes:` pointing at a behavior path that doesn't
  exist — is also E1.

### E2 — `@` prefix used in a bundle-loader YAML field
- **Detect:** `@namespace:path` appearing inside a YAML `context.include:`,
  `agents.include:`, or any `includes:` entry. The `@`-mention form is for the
  **markdown body**; in these bundle-loader fields the correct form is
  `namespace:path` (no `@`).
- **Detect command:** `grep -rnE '^\s*-\s*"?@' behaviors/ *.md`-style scan of YAML
  sections.
- **Scope — do NOT flag these (they are correct):**
  - `@` *inside markdown prose* (e.g. an agent body, a mode body, README).
  - `@` inside a **module's own `config:` block** (e.g. `hooks-mode`'s
    `config.search_paths: [ "@ns:modes" ]`). Those are resolved by that module's
    own @mention logic at runtime, NOT by the bundle loader — a different channel.
    The bundle author should state this intent in a comment; when they have, it's clean.
- **Authority:** BUNDLE_GUIDE.md § *Anti-Patterns*; URI_FORMATS.md.

### E3 — `amplifier-core` (or a sibling bundle) as a module runtime dependency
- **Detect:** any `modules/**/pyproject.toml` lists `amplifier-core` or
  `amplifier-bundle-*` / `amplifier-module-*` under `[project] dependencies`.
  The kernel is the runtime, not a pip dependency of the brick.
- **Authority:** BUNDLE_GUIDE.md § *Anti-Patterns*.

### E4 — `mount()` that does nothing (no-op stub)
- **Detect:** a module `__init__.py` defines `mount(coordinator, config)` but the body
  never actually registers anything with the coordinator — a stub that fails protocol
  compliance. The *correct* registration call is module-type-specific, so check for the
  right one before flagging:
  - **tool / orchestrator / context / provider** modules → `coordinator.mount("tools", ...)`
    (or the matching kind).
  - **hook** modules → `coordinator.hooks.register(...)` (returning a cleanup callable).
  A `mount()` that calls **neither** is the violation. A hook module that registers
  handlers is **clean** — do not flag it for "missing `coordinator.mount()`."
- **Capability-only carve-out (clean, NOT a violation):** a module typed `hook` whose
  `mount()` does real work via `coordinator.register_capability(...)` (and other modules
  consume that capability) but never calls `hooks.register(...)` is **clean** — it is
  doing real work, not a no-op. `register_capability(...)` is a documented coordinator
  method (see core `CONTRACTS.md § mount()`). Only flag a `mount()` that registers
  **nothing at all** (no `mount`, no `hooks.register`, no `register_capability`).
- **Authority:** BUNDLE_GUIDE.md § *mount() Contract*; core `CONTRACTS.md § mount()`;
  `creating-amplifier-modules` skill.

### E5 — `[tool.uv.sources]` used to wire shared code in a module
- **Detect:** `[tool.uv.sources]` with path/git entries in a `modules/**/pyproject.toml`.
  These are silently stripped at runtime install — the wiring you think you have
  isn't there.
- **Authority:** BUNDLE_GUIDE.md § *Anti-Patterns*.

### E6 — Namespace vs. repo-name confusion in includes
- **Detect:** an internal `includes:`/`agents.include:` reference uses the **git
  repo name** (`amplifier-bundle-foo:...`) where it should use the registered
  **bundle name** (`foo:...`), or vice-versa for the self-namespace.
- **Authority:** CONCEPTS.md § *Namespace Registration*; URI_FORMATS.md.

---

## WARNING — anti-pattern that works today but costs later

### W1 — Fat bundle: redeclaring what foundation already provides
- **Detect:** when `amplifier-foundation` is in `includes:`, the bundle/behavior
  also declares core tools (`tool-filesystem`, `tool-bash`, `tool-search`/grep),
  `session:`, or core UI/streaming hooks. Foundation already gives you these.
- **Authority:** BUNDLE_GUIDE.md § *Thin Bundle Pattern*.

### W2 — Capability inline in `bundle.md` instead of a behavior
- **Detect:** substantial `agents:`, `tools:`, `hooks:`, or `context:` blocks living
  directly in the root `bundle.md` frontmatter rather than in a `behaviors/*.yaml`.
  Inline capability isn't composable onto other bundles.
- **Authority:** BUNDLE_GUIDE.md § *Behavior Pattern*.

### W3 — Heavy doc pinned always-on (context-sink violation)
- **Detect:** a file in `behavior.context.include:` that is large (rule of thumb
  **>500 tokens ≈ >2 KB**; **>1 KB tokens** is a strong signal). Heavy docs belong
  in the *agent that needs them* (loaded on demand via `@mention`), not always-on.
- **Detect command:** `wc -c` each included context file; flag the large ones.
- **Authority:** BUNDLE_GUIDE.md § *context.include Policy* / § *Context De-duplication*.

### W4 — Provider declared in the bundle
- **Detect:** a `providers:` / model-provider declaration baked into `bundle.md` or a
  behavior, rather than left to app-layer injection (or isolated in a standalone
  `providers/*.yaml` variant). Pins users to your model choice.
- **Authority:** BUNDLE_GUIDE.md § *Thin Bundle Pattern* / § *Anti-Patterns*.
- **Exception:** a deliberately standalone, runnable bundle variant — check the
  `bundle.md` body for that stated intent before flagging.

### W5 — Spurious root `pyproject.toml`
- **Detect:** a root-level `pyproject.toml` with no legitimate reason — i.e. the
  bundle neither shares code across multiple `modules/` nor ships a standalone CLI.
- **Authority:** BUNDLE_GUIDE.md § *Anti-Patterns*.

### W6 — Instructions duplicated in the bundle.md body
- **Detect:** large blocks of agent/usage instructions inlined in the `bundle.md`
  markdown body that should live in `context/instructions.md` (or the awareness
  file) and be included once.
- **Authority:** BUNDLE_GUIDE.md § *Context De-duplication*.

### W7 — Orphaned context file (dead weight / context-poison risk)
- **Detect:** a `context/*.md` (or `docs/*.md` shipped as bundle context) that is
  referenced by **nothing**: not in any `behavior.context.include:`, not `@mention`-ed
  in any `bundle.md` / `agents/*.md` / `modes/*.md` body, and not a soft-referenced
  read target. It loads for no one — pure dead weight, and a latent W3 if someone
  later wires it in by mistake.
- **Detect method:** for each file under `context/`, grep its basename across the repo
  (`grep -rn "<basename>" behaviors/ agents/ modes/ *.md`). Zero hits outside the file
  itself = orphaned. **Count a file as WIRED if it is `@mention`-ed inside an agent
  body** — that is the correct context-sink pattern (loaded on demand), NOT an orphan.
- **Severity:** WARNING when the orphan is large (**>1 KB tokens ≈ >4 KB**, latent W3) or
  there are several; otherwise INFO. Recommend `git rm` (recoverable via history).
- **Authority:** LANGUAGE_PHILOSOPHY.md § *Dead code is context poison*;
  CONTEXT_POISONING.md § *Context Sink Pattern*; BUNDLE_GUIDE.md § *context.include Policy*.

---

## INFO — polish and discoverability

### I1 — Agent `meta.description` missing "what / when-to-use / usage"
- **Detect:** an `agents/*.md` whose `meta.description` doesn't say what it does AND
  when to reach for it. Description quality drives whether the delegator picks it.
- **Authority:** BUNDLE_GUIDE.md § *Agent Authoring* (and AGENT_AUTHORING.md if present).

### I2 — Empty/weak `bundle.description` or behavior description
- **Detect:** missing, one-word, or placeholder `description:` on the bundle or behavior.

### I3 — Repo hygiene: missing `README.md` or `AGENTS.md`
- **Detect:** no non-trivial `README.md`, or no `AGENTS.md` with test commands /
  gates / pitfalls.
- **Authority:** PER_REPO_CONVENTIONS.md.

### I4 — Stale or missing `bundle.dot`
- **Detect:** no `bundle.dot`, or its `source_hash` no longer matches the repo (the
  doc diagram is stale). Note it; `validate-bundle-repo` can regenerate it.
- **Authority:** `bundle-to-dot` skill; BUNDLE_GUIDE.md § *Documentation*.

---

## Legitimate exceptions (do not flag when stated)

- Root `pyproject.toml` **when** code is genuinely shared across modules or a CLI ships.
- `providers:` **when** the bundle is an intentionally standalone runnable variant.
- Heavy always-on context **when** it truly is needed on every turn (rare — the bundle
  should say so).
- A novel composition the rubric doesn't cover → **Open question → foundation-expert**,
  not a manufactured finding.
