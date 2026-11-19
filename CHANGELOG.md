# Changelog

All notable changes to this project are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `speckit.deliberate.recap` gains a `simple` mode (read-only, no codes or
  paths, short, written for junior engineers).
- `speckit.deliberate.autopilot` gains a `preview` / `dry run` input that
  runs Steps 0–2 and shows the stage plan and would-be writes without
  writing anything, and an `autopilot.chat_report` setting (`brief` default,
  `full`) — chat now prints a short digest while `autopilot-report.md`
  keeps the full run section.
- `speckit.deliberate.autopilot` gains a stage range (`until` / `from` /
  `only` a stage; verdict `READY THROUGH <stage>` when analyze is out of
  range) and an optional triage pass (`autopilot.triage_pass`, or `skip
  triage` per run; the verdict is capped at `READY WITH PENDING DECISIONS`
  while skipped).

### Changed

- `speckit.deliberate.auto` renamed to `speckit.deliberate.autopilot`
  (BREAKING): command file, `auto:` config block → `autopilot:` (the
  `mode: auto` detect value is unchanged), report file `auto-report.md` →
  `autopilot-report.md`, guide `docs/auto-mode.md` → `docs/autopilot.md`.
  Migrate: rename your `auto:` block and rename existing `auto-report.md`
  files.

## [0.4.0] - 2026-11-18

### Added

- `speckit.deliberate.auto` — gated pipeline after explore or recap: a
  read-only context sweep (constitution, feature tree, DRs, repo guidance,
  code) builds an evidence pack and a context tier, then specify → clarify →
  plan → tasks → analyze run with clarify re-entering on any detected gap and
  resuming at the earliest affected stage. Gaps are ranked provisionally
  with triage's signals and resolved by evidence, a tagged balanced default,
  or left open; before the verdict a read-only `triage` pass (`all full`)
  sets final ranks and merges uncertainties already in the tree — existing
  DRs keep their ids, everything else becomes a non-approvable open item
  routed to its owning command. DRs go through record: written `Accepted`
  only under user approval, else queued in the report as `PD-N` with no DR
  file or ledger row; approvals re-check evidence and DR-worthiness;
  supersession is queued, then confirmed; dependent edits never applied
  early. A named change ("change DR-NNN to …") bypasses the contradiction
  gate into that confirmation path. Bounded analyze → remediate loop,
  iterate mode that approves or rejects queued items and
  reconciles every active artifact after a changed decision, and a verdict
  (`READY` · `READY WITH PENDING DECISIONS` · `BLOCKED`) appended to
  `auto-report.md`. Never implements or commits.
- `auto:` block in `config-template.yml` (all keys optional with defaults)
  and `docs/auto-mode.md` guide.

### Changed

- README: install and update docs state that every `specify extension`
  command runs from the consumer project's root (or `SPECIFY_INIT_DIR`), and
  a new Update section covers `--force --from` reinstalls and config
  preservation.

## [0.3.0] - 2026-09-16

### Added

- `speckit.deliberate.triage` — read-only uncertainty queue: harvests
  cross-artifact gaps, ranks P0/P1/P2 with `blocks X` citations, groups by
  fixed category slugs (core taxonomy + deliberate-only), and routes each
  head item to `clarify` / `record` / ADR path. Queue-only: never runs a
  resolution loop.

## [0.2.0] - 2026-09-16

### Added

- `speckit.deliberate.recap` — read-only convergent status brief: state
  (`exploration` / `in-progress` / `done`), decided items, open questions with
  recommended + alternative options, draft epic/issue/PR titles, ordered next
  steps, and a portable agnostic TL;DR.

### Changed

- README: install docs clarify the `community` catalog is discovery-only
  (`install_allowed: false`); header and Why table updated for three commands.

## [0.1.1] - 2026-09-16

### Changed

- Personal-contribution sanitization: removed all prior-org references.
  The worked platform overlay is now a fully fictional example
  (`docs/platform-overlay-example.md` with invented Acme/SnapBus/PolicyGraph
  names). Leak-lint enforces the fictional terms instead of any real stack.
- Install docs now point at the release asset
  (`releases/download/.../deliberate-vX.Y.Z.zip`) with a SHA-256 verify note
  and a private-repo `~/.specify/auth.json` hint.
- Raise `requires.speckit_version` floor `>=0.2.0` → `>=1.0.0` (smoke-tested
  on CLI 1.0.7; tokens/templates need a modern CLI).

### Fixed

- `explore` and `record` now document PowerShell variants alongside Bash for
  `check-prerequisites` and DR-number allocation.

## [0.1.0] - 2026-09-15

### Added

- `speckit.deliberate.explore` — read-only thinking partner (stance,
  guardrails, progressive context load, options tables, capture offers).
- `speckit.deliberate.record` — feature Decision Records with classify
  (DR/ADR/SKIP), `DR-NNN` allocation, template scaffolding, `spec.md` ledger,
  supersede-not-delete, batch mode, and chat-only fallback without a feature.
- `decision-record-template` — generic 10-section DR template with evidence
  bar and revisit trigger.
- `deliberate-config.yml` — decisions dir, ledger heading, task default,
  evidence rule, and `platform_routing` overlay (ADR command + signals).
- Docs: capture-routing, ledger, routing, template-checklist references plus
  a worked platform-overlay example.
- CI: extension validation (manifest, tokens, links, leak-lint), DCO,
  license-header, and YAML lint.
