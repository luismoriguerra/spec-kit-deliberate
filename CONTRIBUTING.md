# Contributing

Thanks for helping improve `deliberate`. This repo ships a Spec Kit
extension, so contributions are mostly prompt-markdown, templates, and docs —
held to the same bar as code.

## Before you push

1. **Sign off every commit (DCO):** `git commit -s`. Enforced by CI.
2. **Keep CI green:** manifest validation, token/link lint, leak-lint, YAML
   lint, license headers. Run the checks locally where you can (see below).
3. **Prove behavior with a `--dev` install** and paste the smoke test into
   your PR (see the PR template).

## Where does my change go?

| Change type | Location |
|---|---|
| Explore/record/recap/triage behavior | `commands/explore.md`, `commands/record.md`, `commands/recap.md`, `commands/triage.md` |
| DR template | `templates/decision-record-template.md` |
| Config surface | `config-template.yml` (+ README sample) |
| Reference docs | `docs/capture-routing.md`, `docs/ledger.md`, `docs/routing.md`, `docs/template-checklist.md` |
| Project-specific worked example | `docs/platform-overlay-example.md` (the ONLY place for it) |

## Generic-core rules (CI-enforced)

The extension must work in any Spec Kit project. Therefore:

1. **No project-specific platform terms** in `extension.yml`, `commands/`,
   `templates/`, `config-template.yml`, or `README.md`. Worked overlays
   belong in `docs/platform-overlay-example.md` only.
2. **No literal command invocations** in `commands/` — no `/speckit.…`
   spellings. Reference siblings via `__SPECKIT_COMMAND_*__` tokens so every
   agent integration renders correctly.
3. **No repo-relative markdown links** in `commands/` — installed skills are
   linted by consumers, and relative links break there. Reference install
   paths as plain code spans
   (e.g. `.specify/extensions/deliberate/docs/ledger.md`).
4. **`docs/` must ship** — never add it to `.extensionignore`; command
   bodies reference it at the install path.

## Versioning

- Follow SemVer in `extension.yml` (`MAJOR.MINOR.PATCH`, no `v` prefix).
- Any change under `commands/` or `templates/` MUST bump the version and
  add a `CHANGELOG.md` entry — updates are version-driven, so unbumped
  content never reaches installed copies.
- Docs-only changes SHOULD still add a changelog line; patch bump optional.

## Local checks

```bash
# YAML parses + required keys present
python3 -c "import yaml,sys; yaml.safe_load(open('extension.yml'))"

# No literal invocations in command bodies
! grep -rn 'speckit\.' commands/ | grep -v '__SPECKIT_COMMAND_'

# No repo-relative links in command bodies
! grep -rn '](docs/\|](\.\./' commands/

# No project-specific terms in the generic core — see the leak-lint step in
# .github/workflows/extension-validate.yml for the exact pattern set

# Smoke test on a scratch project
specify extension add --dev "$PWD"
specify extension list
specify extension remove deliberate
```

## PR process

- One concern per PR; keep diffs reviewable.
- Fill the PR template (summary, test plan, semver impact).
- A CODEOWNERS review is required; only maintainers tag releases (`vX.Y.Z`
  on `main`).
- Catalog updates happen via a new Extension Submission issue after
  release — never by editing catalog JSON in this repo.

## Commit style

Conventional commits (`feat: …`, `fix: …`, `docs: …`, `chore: …`) with a
DCO `Signed-off-by` trailer.
