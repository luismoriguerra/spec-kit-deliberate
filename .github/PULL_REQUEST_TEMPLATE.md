<!--
Deliberate extension PR. All commits must be DCO signed off (git commit -s).
-->

## Summary

<!-- What & why, 1–3 bullets. -->

## Test plan

- [ ] `--dev` smoke test on a scratch project (`add` → `list` → exercise commands → `remove`) — paste output
- [ ] Local checks from CONTRIBUTING.md pass (manifest parses, token/link lint clean)
- [ ] Commits are DCO signed off (`git commit -s`)

## Semver impact

- [ ] No change under `commands/` or `templates/` (no bump needed)
- [ ] Patch / Minor / Major — `extension.yml` version bumped + `CHANGELOG.md` entry added

## Docs updated

- [ ] README / docs reflect the behavior change (or N/A)
- [ ] Generic core stays project-agnostic (no platform-specific terms outside `docs/platform-overlay-example.md`)
