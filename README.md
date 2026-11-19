# Deliberate — Spec Kit extension

**Think before you spec. Record what you decided.**

`deliberate` adds five commands around Spec Kit's
specify → clarify → plan → tasks → implement flow — four that think and
record, one that drives the flow:

- **explore** (`speckit.deliberate.explore`) — a read-only thinking partner:
  brainstorm, compare approaches, and investigate the codebase without editing
  specs, writing code, or auto-capturing anything.
- **record** (`speckit.deliberate.record`) — feature Decision Records (DRs):
  classify scope, allocate `DR-NNN`, scaffold from a template, keep a
  `spec.md` ledger, and supersede (never delete) prior decisions.
- **recap** (`speckit.deliberate.recap`) — a read-only status brief:
  state, decided items, open questions with recommendations, draft
  epic/issue/PR titles, next steps, and an agnostic TL;DR. Supports a `simple` mode (`recap simple`).
- **triage** (`speckit.deliberate.triage`) — a read-only uncertainty queue:
  harvests cross-artifact gaps, ranks P0/P1/P2, groups by category, and
  routes each head item to its owning capture command.
- **autopilot** (`speckit.deliberate.autopilot`) — a gated pipeline: sweeps the project
  for constraints and evidence, runs specify → clarify → plan → tasks →
  analyze, settles recommendations through DRs, applies safe remediations,
  and reports whether the spec is ready for review. Iterate runs reconcile
  every document after a changed decision.

**Version:** 0.5.0 · **Requires:** spec-kit ≥ 1.0.0 · **License:** MIT
**Provides:** 5 commands, 1 template · **Hooks:** none in v1

---

## Why

Spec Kit drives **spec → code** (forward). These gaps stay open:

| Without it | With it |
|---|---|
| "Should we…" debates happen in chat and evaporate | explore compares options read-only, then offers one explicit capture path |
| Decisions hide inside plan prose nobody re-reads | each decision is a `DR-NNN` file with evidence and a revisit trigger |
| Old choices get silently rewritten | supersede-not-delete keeps history: new DR + updated ledger + old body intact |
| Feature call vs platform rule is a judgment call every time | the DR-vs-ADR firewall routes platform scope to your ADR workflow |
| Analysis findings rot in reports | record materializes them as DRs with tasks |
| No shared view of decided vs open vs next | recap converges the tree into a state line + five-section brief |
| Uncertainties pile up with no order to kill them | triage ranks P0/P1/P2 by impact, groups by category, routes each head item |
| Specify, clarify, plan, tasks, analyze get chained by hand after every exploration and every changed decision | autopilot runs them gated by evidence and approval, then reports readiness |

## How it works

```
explore (read-only)  →  (user decides)  →  clarify / record / plan  →  …
explore (read-only)  →  autopilot (specify → clarify → plan → tasks → analyze)  →  review  →  implement
```

1. **Explore** loads only what the conversation needs (`spec.md`, DRs,
   `tasks.md`, source tree), compares ≥2 approaches, and sketches DR-worthy
   questions — then stops. Capture is always offered, never automatic.
2. **Record** classifies the question (DR / ADR / SKIP), allocates the next
   `DR-NNN`, drafts from the canonical template, confirms the plan, then
   writes the file + ledger row (+ optional task).
3. Without an active feature, record degrades to a **chat-only proposal**
   instead of writing files — it never creates `specs/` entries itself.
4. **Recap** converges the tree into a five-section brief — Decided, Open
   (with Recommended + alternatives), draft epics/issues/PRs, What is next,
   and an agnostic TL;DR — with a state line (`exploration` / `in-progress`
   / `done`). Read-only, like explore.
5. **Triage** harvests every uncertainty across the tree into one visible
   queue — board ranked P0/P1/P2, grouped by fixed category slugs, each
   head item routed to `clarify` / `record` / ADR path. Queue-only,
   read-only; re-run with a scope filter to continue by category.
6. **Autopilot** sweeps the constitution, feature tree, DRs, repo guidance, and
   code into a Context Pack, then runs the core stages. Every gap is ranked
   P0/P1/P2 and resolved with evidence, a tagged balanced default, or left
   open. Evidenced DRs are written `Accepted` only under your approval;
   otherwise they stay queued in the report (`PD-N`) with nothing written.
   Superseding always waits for confirmation. Clarify re-runs whenever a
   stage finds a gap, resuming at the earliest affected stage. Before the
   verdict, a read-only triage pass (`all full`) merges uncertainties already
   in the tree. Ends with a verdict (`READY` · `READY WITH PENDING DECISIONS`
   · `BLOCKED`) appended to `autopilot-report.md`. Never implements or commits —
   see `docs/autopilot.md`.

### The DR contract

Every DR carries: Status · Context + single Question · Options Considered ·
Assumptions · Evidence · Decision · Confidence · Consequences · Revisit
Trigger. Hard fails: empty Evidence/Revisit Trigger, multi-question records,
Accepted-without-evidence, platform scope written as DR.

The `decisions/` directory has no index — the canonical index is the
`Decision records` table in `spec.md`, kept in numeric order.

## Install

Run every `specify extension` command from the root of the Spec Kit project
that consumes the extension (the directory containing `.specify/`), or set
`SPECIFY_INIT_DIR` to it. The CLI installs into that project — not into the
source path or URL you pass.

Pre-catalog (from the tagged release asset):

```bash
cd /path/to/consumer-spec-kit-project
specify extension add deliberate --from https://github.com/luismoriguerra/spec-kit-deliberate/releases/download/v0.4.0/deliberate-v0.4.0.zip
# Verify out-of-band: compare sha256sum of the zip with the SHA-256 in the release notes
```

Local development install (from a checkout of this repo, into a consumer or
scratch Spec Kit project):

```bash
cd /path/to/consumer-spec-kit-project
specify extension add --dev /path/to/spec-kit-deliberate
```

Verify:

```bash
specify extension list
# ✓ Deliberate (v0.4.0) — Commands: 5 | Hooks: 0 | Status: Enabled
```

On skills-based integrations the commands also appear as agent skills
(`speckit-deliberate-explore`, `speckit-deliberate-record`,
`speckit-deliberate-recap`, `speckit-deliberate-triage`,
`speckit-deliberate-autopilot`).

Once in the official default catalog or your own install-allowed catalog:

```bash
specify extension add deliberate
```

> Listing in the `community` catalog does NOT enable this: it is
> discovery-only (`install_allowed: false`). From there the flow stays
> `specify extension info deliberate` → `specify extension add deliberate --from <candidate-archive-url>`.

## Update

From the consumer project's root, reinstall over the existing copy:

```bash
cd /path/to/consumer-spec-kit-project
specify extension add deliberate --force --from https://github.com/luismoriguerra/spec-kit-deliberate/releases/download/v0.4.0/deliberate-v0.4.0.zip
specify extension list   # ✓ Deliberate (v0.4.0) — Commands: 5
```

- `specify extension update deliberate` only works once deliberate is in an
  install-allowed catalog; until then the CLI skips it. Use `--force --from`.
- `--force` keeps your `deliberate-config.yml` (the CLI backs up and restores
  `*-config.yml`) but does not merge new keys. New settings are optional with
  defaults; copy new blocks (e.g. `autopilot:`, renamed from `auto:` in 0.5.0) from `config-template.yml`
  to customize.
- Feature trees (`specs/`, DRs, ledgers) are never touched by an update.

## Adopt

```bash
# 1. Optional: copy the config template and set your platform overlay
cp .specify/extensions/deliberate/config-template.yml \
   .specify/extensions/deliberate/deliberate-config.yml

# 2. Before writing a spec — think out loud, change nothing
explore the tradeoffs for <idea>

# 3. After a decision is chosen — record it with evidence
record a DR for <question>

# 4. Anytime — converge the tree into a status brief
recap this feature

# 5. Anytime — rank uncertainties by impact and category
triage this feature

# 6. After explore converges — run the gated pipeline, get a readiness report
autopilot this feature

# 7. After a recap or a changed decision — reconcile for the next review
autopilot: change DR-004 to <new option>
```

No active feature yet? Record drafts the proposal in chat and tells you to
run specify first — nothing is written until a feature exists. Autopilot runs
specify itself when the conversation holds an explore brief.

## Commands & configuration

| Command | Effect | What it does |
|---|---|---|
| `speckit.deliberate.explore` | read-only | Thinking partner: options, tradeoffs, risks, capture offers |
| `speckit.deliberate.record` | read-write | Classify → allocate → draft → confirm → write DR + ledger |
| `speckit.deliberate.recap` | read-only | Status brief: decided, open + recommendations, epics/PRs, next, TL;DR |
| `speckit.deliberate.triage` | read-only | Uncertainty queue: P0/P1/P2 board, categories, routed resolution queue |
| `speckit.deliberate.autopilot` | read-write | Context sweep → specify → clarify → plan → tasks → analyze → remediate → readiness report; iterate reconciles changes |

`deliberate-config.yml` (all optional, defaults shown):

```yaml
decisions_dir: decisions
ledger_heading: "### Decision records"
task_enabled: true
evidence_rule: ">=1 file:line or measured query result"
platform_routing:
  adr_command: ""       # e.g. your ADR review/updater command
  platform_signals: []  # project-specific ADR triggers
```

Autopilot adds an optional `autopilot:` block (approval, default policy, loop
bounds, report file) — see `docs/autopilot.md`. See
`docs/platform-overlay-example.md` for a worked platform-overlay example.

## Not to be confused with

- **adrkit** — pulls platform ADRs into agent context and drafts ADRs from
  plans (platform scope; this extension handles feature scope).
- **arch-governance** — keeps specs, code, and ADRs in sync with citation
  slots + a validator (integrity checking; this extension authors DRs).
- **discovery** — feasibility and technology selection (pre-spec research;
  this extension deliberates options and records decisions).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) — DCO sign-off required, CI must be
green, generic core stays project-agnostic.

## License

MIT — see [LICENSE](LICENSE).
