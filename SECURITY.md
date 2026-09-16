# Security Policy

## Supported versions

| Version | Supported |
|---|---|
| 0.1.x | Yes (pre-release; fixes land on `main` and the next tag) |

## Reporting a vulnerability

Do **not** open a public issue for a suspected vulnerability.

- Prefer a **private GitHub Security Advisory** on this repository
  (Security → Advisories → Report a vulnerability).
- If advisories are unavailable, contact a maintainer listed in
  CODEOWNERS directly.

Include: affected version/commit, a minimal reproduction, and the impact
you see. Redact secrets, tokens, and private repo paths.

## Response targets

- Acknowledgement within 3 business days.
- Fix or mitigation plan within 14 days for confirmed issues.

This extension runs inside the user's agent session with their own
credentials — it must never exfiltrate data, and contributions must not
add network calls, credential handling, or obfuscated instructions.
