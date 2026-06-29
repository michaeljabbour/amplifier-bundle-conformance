# The Amplifier-Way Conformance Rubric

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

### E2 — `@` prefix used in a YAML field
- **Detect:** `@namespace:path` appearing inside a YAML `context.include:`,
  `agents.include:`, or any `includes:` entry. The `@`-mention form is for the
  **markdown body**; in YAML the correct form is `namespace:path` (no `@`).
- **Detect command:** `grep -rnE '^\s*-\s*"?@' behaviors/ *.md`-style scan of YAML
  sections. (`@` is correct *inside markdown prose*, e.g. an agent body — don't
  flag those.)
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
- **Authority:** BUNDLE_GUIDE.md § *mount() Contract*; `creating-amplifier-modules` skill.

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
