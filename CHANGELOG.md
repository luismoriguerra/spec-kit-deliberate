# Changelog

All notable changes to this project are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
