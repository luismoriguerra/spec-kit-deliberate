---
description: "Run specify → clarify → plan → tasks → analyze, gated by evidence and approval"
---

# Autopilot

**Gated pipeline for Spec Kit** — the one deliberate command that drives the
core flow. After explore converges, or after a recap names a change, autopilot
grounds itself in project evidence, runs specify → clarify → plan → tasks →
analyze, settles recommendations through decision records, applies safe
remediations, and reports whether the spec is ready for review.

> **Evidence first, authority second.** Autopilot verifies options on its own; it
> accepts decisions only under an approval the user gave. Everything else is
> queued, ranked, and reported — never treated as decided.

**Writes:** inside the feature directory only — `spec.md`, `plan.md`,
`research.md`, `data-model.md`, `contracts/`, `quickstart.md`, `tasks.md`,
`checklists/`, `DR-*.md` files under the configured decisions dir, the
Decision records ledger, and the run report (`autopilot.report_file`, default
`autopilot-report.md`). With no feature yet: whatever the specify stage creates
(`specs/NNN-*`, `.specify/feature.json`, a branch if a `before_specify` hook
creates one). `.specify/feature.json` also changes when a pinned stage
persists the pinned feature (Step 0).

**Does not write:** application code, the constitution, ADR logs, other
features, extension files, or git history. Never runs
`__SPECKIT_COMMAND_IMPLEMENT__`.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

Parse optional arguments:

| Input | Default |
|---|---|
| Feature | Active feature (`check-prerequisites.sh`) or named `specs/NNN-*`; none → greenfield (Step 0) |
| Mode | `autopilot.mode` (`auto`) · `initial` · `iterate` |
| Change | Iterate only — a decision change ("change DR-004 to event-driven"), approvals ("approve PD-3, PD-5"), or a requirement change |
| Approval | `autopilot.approval` (`queue`); "just write it" in the input means `accept-evidenced` for this run |
| Depth | `autopilot.sweep_depth` (`standard`) · `quick` · `deep` |
| Preview | `preview` / `dry run` in the input → run Steps 0–2 only and show the plan; writes nothing (see "Preview") |
| Report | `autopilot.chat_report` (`brief`) · `full` — what Step 9 prints in chat; the report file always gets the full run section |
| Range | `until <stage>` / `through <stage>` · `from <stage>` · `only <stage>` (stages: specify, clarify, plan, tasks, analyze); default all — see "Stage range" (Step 4) |
| Triage pass | `autopilot.triage_pass` (`true`); `skip triage` / `no triage` in the input skips it for this run (Step 8) |

Read `.specify/extensions/deliberate/deliberate-config.yml` when present:
`decisions_dir`, `ledger_heading`, `task_enabled`, `evidence_rule`,
`platform_routing`, and the `autopilot:` block. Every `autopilot.*` key is optional;
a missing block or key takes the default documented in
`.specify/extensions/deliberate/config-template.yml`.

## Triggers

Follow this command when the user message matches **any** of these (case
insensitive; partial match is fine):

| Trigger | Example |
|---|---|
| `autopilot` / `autopilot mode` | "autopilot this feature" |
| run the pipeline | "run specify through analyze for this idea" |
| after explore | "we're done exploring — autopilot it" |
| change a decision | "autopilot: change DR-004 to event-driven" |
| approve pending | "autopilot: approve PD-3 and PD-5" |
| reconcile / ready for review | "reconcile the docs for the next review" |
| preview / dry run | "autopilot preview", "autopilot dry run for this feature" |
| stage range | "autopilot until plan", "autopilot from tasks", "autopilot only analyze" |

**Not a trigger** — redirect instead:

| User intent | Command |
|---|---|
| Still brainstorming | `__SPECKIT_COMMAND_DELIBERATE_EXPLORE__` |
| One decision only | `__SPECKIT_COMMAND_DELIBERATE_RECORD__` |
| Status brief only | `__SPECKIT_COMMAND_DELIBERATE_RECAP__` |
| Ranked queue only | `__SPECKIT_COMMAND_DELIBERATE_TRIAGE__` |
| Read-only audit only | `__SPECKIT_COMMAND_ANALYZE__` |
| Implement code | `__SPECKIT_COMMAND_IMPLEMENT__` — autopilot never runs it |

## Hard guardrails (non-negotiable)

- **No code, no implement, no commits** — autopilot stops at a reviewable spec
  tree. No git commit/push, build, migration, or deploy.
- **Feature-scoped writes** — edit only inside `FEATURE_DIR`, plus what the
  specify stage creates. Never edit the constitution, ADR logs, other
  features, or extension files.
- **Evidence ≠ authority** — verification runs unattended; acceptance does
  not. No DR file or ledger row is written unless Step 3's approval rule
  covers it. Superseding an Accepted DR always needs the user's explicit
  confirmation of the queued draft (Step 7).
- **Never act on a pending decision as if accepted** — remediation and
  reconcile edits that depend on a queued decision, an existing Proposed DR,
  a default, or an unconfirmed supersession are not applied.
- **Defaults are assumptions** — a balanced default is tagged with
  `autopilot.mark_assumed`, never counts toward `evidence_rule`, and never becomes
  an Accepted DR.
- **Platform scope is not a DR** — platform signals go to the ADR handoff in
  the routing reference, take no P-rank, and are never defaulted.
- **Adopt, don't regenerate** — an existing artifact is revised in place,
  section by section; never regenerated over the user's content.
- **History is immutable** — old DR bodies, superseded ledger rows,
  clarification session bullets, and prior report runs stay intact.
- **Bounded** — `autopilot.clarify_max_interrupts` and `autopilot.analyze_max_passes`
  cap every loop; the same gap never triggers twice.
- **Grounded** — every edit, decision, and default cites its source:
  `file:line`, a measured result, `conversation`, or `default: balanced`.

---

## Step 0 — Resolve feature, mode, and config

From the project root, run the appropriate script for your platform:

```bash
.specify/scripts/bash/check-prerequisites.sh --json --paths-only
```

```powershell
.specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
```

Parse `FEATURE_DIR` and `FEATURE_SPEC`; read `.specify/feature.json` when the
script is not enough. Set `DECISIONS = $FEATURE_DIR/<decisions_dir>` and
`REPORT = $FEATURE_DIR/<autopilot.report_file>`.

**Pin the feature.** A feature named in the input wins over the active one:
resolve it by running the script with `SPECIFY_FEATURE_DIRECTORY=<named
path>` (PowerShell: `$env:SPECIFY_FEATURE_DIRECTORY`), Spec Kit's own
override. Either way — and for greenfield, from the directory specify
creates — `FEATURE_DIR` holds for the whole run:

- Run every script call — yours and each stage's — with the same override.
- Pass the feature path to every delegated command (`specs/NNN-* …` as its
  input).
- After each stage, confirm it resolved and wrote under `FEATURE_DIR`; any
  other target stops the run — `BLOCKED — feature mismatch (<got> ≠
  <FEATURE_DIR>)`.
- Core scripts may persist the override to `.specify/feature.json`; when the
  active feature changed, say so in the report.

| Situation | Action |
|---|---|
| Feature with `spec.md` | Use it; the specify stage reports `skipped (spec exists)` |
| No feature, explore brief in this conversation, `autopilot.run_specify: true` | Greenfield — the specify stage creates the feature (Step 4) |
| No feature, `autopilot.run_specify: false` | Stop — `BLOCKED: needs specify` |
| No feature, no explore brief | Stop — run `__SPECKIT_COMMAND_DELIBERATE_EXPLORE__` first; never invent a feature description |

The **explore brief** is what the user said in this conversation: the
problem, the chosen direction, options ruled out, and stated limits. A lean
the agent offered is not the user's choice.

Mode when `autopilot.mode` is `auto`: `iterate` if `REPORT` holds a prior run or a
Change input is present; otherwise `initial`. `initial` on an existing
feature adopts its artifacts (Step 4).

---

## Step 1 — Context sweep → Context Pack (read-only)

Gather everything that constrains or evidences the work before any stage
runs. `autopilot.sweep_depth`: `quick` = rows 1–5; `standard` = + codebase search
for the feature's domain terms; `deep` = + dependency manifests, tests, and
`git log` of the touched areas.

| # | Source | Yields |
|---|---|---|
| 1 | `.specify/memory/constitution.md` | Principles and MUST rules — hard constraints |
| 2 | Explore brief / recap in this conversation | Intent, non-goals, rejected options, stated limits |
| 3 | Feature tree: `spec.md`, `plan.md`, `research.md`, `data-model.md`, `contracts/`, `tasks.md`, `checklists/` | Scope, requirements, tradeoffs already made |
| 4 | `$DECISIONS/DR-*.md` + ledger | Prior choices, Revisit Triggers — never re-decide an Accepted DR silently |
| 5 | Repo and agent guidance (`AGENTS.md`, agent instruction files, `README`, `CONTRIBUTING`, `docs/`), `autopilot.context_sources`, `platform_routing` + project ADR log | Conventions, helpers, limits, platform rules |
| 6 | Codebase (read-only search) | Existing helpers, patterns, APIs, tests — `file:line` evidence, reuse over reinvention |
| 7 | Runtime | Platform, available scripts and tools, Spec Kit version — what this run can actually do |

Tag each pack item with a type, a source, and a strength (`evidenced` ·
`stated` · `inferred`):

| Type | Meaning | Use |
|---|---|---|
| Constraint | Constitution MUST, platform rule | Violations stop the run (Step 2) |
| Restriction | User or system limit ("no new branch", no network) | Shrinks the pipeline; autopilot never exceeds it |
| Intent | Goal, non-goals, ruled-out options | Specify input; tie-breaker for recommendations |
| Knowledge | Helpers, patterns, prior DRs, docs | Evidence for decisions and plan |
| Assumption | Inferred, unverified | Listed in the report; never the sole basis of a DR |

**Context tier** (reported, never hidden):

| Tier | Signal |
|---|---|
| `Rich` | Constitution, feature artifacts or DRs, and code evidence for the feature area |
| `Partial` | Some sources present, some missing |
| `Sparse` | No constitution and no code or doc evidence for the area (typical greenfield) |

Missing sources are recorded as `none found`, never guessed. With
`autopilot.require_constitution: true` and no constitution, stop — `BLOCKED`.
Re-sweep the delta after specify and after plan (new artifacts only).

---

## Step 2 — Gate (before any write)

Stop with `BLOCKED` and cite both sides when:

- Two sources contradict — constitution vs explore direction, an Accepted DR
  vs stated intent, a restriction vs a stage the run needs.
- The intended direction violates a constitution MUST.

A Change input that **names** the decision it replaces ("change DR-004 to
…") is not a contradiction: route it to Step 7's decision-change intake,
whose confirmation gate holds every dependent edit. A conflict nobody named
stops the run; the report suggests stating it as a change
(`__SPECKIT_COMMAND_DELIBERATE_AUTOPILOT__ change DR-NNN to …`).

Otherwise continue. Autopilot never resolves a contradiction by picking a side.

---

## Preview — plan without writing

When the input says `preview` or `dry run`: run Steps 0–2, then stop. Write
nothing — no artifact, no DR, no ledger row, no `REPORT` section, no
`.specify/feature.json` change. Run only the read-only `--paths-only` script
(no stage commands, no triage pass, no subagents).

Emit one block, then wait:

```markdown
**Preview — <feature or greenfield>** · mode: initial|iterate · range: all|<range> · tier: Rich|Partial|Sparse
**Gate:** clear | BLOCKED — <both sides of the conflict>

| Stage | Would do | Why |
|---|---|---|
| specify | skip (spec exists) / generate / skipped (range) / not run: out of range | … |
| clarify | run / skipped (range) / not run: out of range | the stage row follows `Range`; gap-driven interrupts can still fire in a run |
| plan | generate / revise <sections> / skipped (range) / not run: out of range / not run: <reason> | … |
| tasks | generate / revise <sections> / skipped (range) / not run: out of range / not run: <reason> | … |
| analyze | run / not run: out of range / not run: no `tasks.md` | runs when in range and `tasks.md` exists |
| triage pass | run / skipped | runs regardless of range unless skipped (`autopilot.triage_pass`, `skip triage`) |

With `from X`, also flag any earlier stage whose required artifact is missing as `BLOCKED — range needs <artifact>`.

**Writes if run:** files under FEATURE_DIR · DR files: none written on autopilot's own judgment under `queue`; `accept-evidenced` allows evidenced, supersede-nothing DRs · a Change with an `approve PD-N` / `approve DR-NNN` would write or flip that DR once it passes Step 7's re-check (the user's message is the approval, even under `queue`) · report: `<REPORT>`
**Known pending (from the last run / sweep):** PD-N and open items, or `none`
**Iterate intake (if a Change was given):** what it would supersede or approve, with the confirmation gate that would hold

To run for real: `__SPECKIT_COMMAND_DELIBERATE_AUTOPILOT__` with the same input, without `preview`.
```

Preview only reports what Steps 0–2 can know; do not predict clarify answers,
analyze findings, or the verdict.

---

## Step 3 — Gap and decision protocol

Every stage routes each question it meets — spec marker, clarify question,
plan decision point, analyze finding — through this protocol instead of
asking the user.

1. **Classify** with the routing reference: requirement gap → spec answer;
   DR-worthy choice → DR; platform-scoped → ADR handoff; SKIP.
2. **Rank provisionally** P0/P1/P2 with a `blocks X` citation, using
   triage's signals: `P0` blocks spec/plan/tasks/implement, or auth/privacy
   with unknown authority or read path; `P1` touches 2+ FR/SC, carries a
   `[high risk]` assumption, or a Revisit Trigger is now true; `P2`
   single-surface, reversible, low blast radius. Conflicting signals take
   the higher rank. Rank before resolving — evidence never lowers a rank.
   Platform signals take no rank. The Step 8 triage pass sets final ranks.
3. **Resolve**, first match wins:
   - **Verified** — the Context Pack yields an option that meets
     `evidence_rule`. For DR-worthy choices, draft it through
     `__SPECKIT_COMMAND_DELIBERATE_RECORD__` in `propose-only` mode (no
     writes).
   - **Defaulted** — no evidence, rank ≤ `autopilot.defaultable_max_rank`, not
     non-defaultable, and `autopilot.default_policy` is not `stop`.
   - **Open** — neither.
4. **Write by authority:**

| Outcome | Requirement gap | DR-worthy choice |
|---|---|---|
| Verified | Answer in `spec.md` (clarify format), cite evidence | Covered by approval → record writes `Accepted`; else queued — no write |
| Defaulted | Answer in `spec.md` tagged `mark_assumed` → Assumptions | Queued — no write (evidence bar unmet) |
| Open | Leave or add `[NEEDS CLARIFICATION: …]` → pending | Queued — no write |
| Supersedes an Accepted DR | — | Queue the draft as a `PD-N` with record's Step 4 plan; dependent edits blocked until confirmed (Step 7) |

**Approval** comes only from the user: `autopilot.approval: accept-evidenced`
(standing approval for new feature-local DRs that meet the evidence bar and
supersede nothing), "just write it" in this run's input, or an explicit
in-thread choice of that option. `queue` (default) writes no DR on autopilot's
judgment.

**Queued** decisions live only in the report's Pending decisions, with the
`propose-only` draft's question, recommendation, alternatives, and evidence
(`none` when the evidence bar is unmet). Each gets a stable `PD-N` id,
numbered across the report's runs, kept until the user approves or rejects
it or a later run resolves it. Only DR-worthy choices get a `PD-N`.

**Non-defaultable** (stay open regardless of policy): auth/privacy without
named protected fact, authority, and read path; data loss, destructive
operations, migrations; security boundaries; legal or compliance calls;
anything the constitution forbids or a restriction reserves to the user;
platform signals.

**Balanced default** (`default_policy: balanced`): list 2–3 options that
satisfy every constraint and restriction; score each on effort, cost (run,
maintenance, dependency weight), and reversibility (`low` · `med` · `high`);
pick the best overall balance, ties to the more reversible. Record the
rejected options and a concrete revisit trigger. `conservative` picks the
lowest-risk, most reversible option; `stop` disables defaults.

**Building on pending items:** later stages may build on a queued or
defaulted option only when it ranks P1/P2, marking each use (`pending PD-N`
or the `mark_assumed` tag). A pending P0 halts every stage it blocks.

---

## Step 4 — Run the pipeline

Each stage is either **generate** (artifact absent → run the core command) or
**revise** (artifact exists → targeted edits to the affected sections,
following that command's template and rules).

Run a core stage the same way you would run the command yourself in this
agent/session (the invocation may differ by agent, e.g. a skills form).
Honor its hooks and its own quality gates. When the stage would ask the user
a question, answer through Step 3 instead; open items stay as markers.

| # | Stage | Command | Input | Continue when |
|---|---|---|---|---|
| 1 | specify | `__SPECKIT_COMMAND_SPECIFY__` | Explore brief + pack intent and constraints | `spec.md` exists; markers routed via Step 3 |
| 2 | clarify | `__SPECKIT_COMMAND_CLARIFY__` | Pack + open markers as prioritization context | No pending P0 blocks plan |
| 3 | plan | `__SPECKIT_COMMAND_PLAN__` | Spec + pack constraints and knowledge | Plan artifacts exist; its Constitution Check passes |
| 4 | tasks | `__SPECKIT_COMMAND_TASKS__` | Plan artifacts | `tasks.md` exists; no pending P0 blocks tasks |
| 5 | analyze | `__SPECKIT_COMMAND_ANALYZE__` | Whole feature tree | Step 6 loop |
| 6 | triage pass | `__SPECKIT_COMMAND_DELIBERATE_TRIAGE__` | `all full` | Step 8 merge, then verdict; skipped when `autopilot.triage_pass: false` or the input says `skip triage` |

**Adopting an existing feature:** a stage whose artifact exists reports
`skipped (exists)` unless Step 5 or iterate intake names an affected
section — then it runs as revise. Within the range, clarify and analyze
always run; the triage pass runs unless skipped (Step 8).

After every stage run the Step 5 detector. Mark a stage `not run: <reason>`
when a pending P0, an awaited ADR handoff, or a gate blocks it.

### Stage range

`Range` limits which stages run. Steps 0–3 (resolve, sweep, gate, protocol)
always run, and a range never widens an approval.

| Input | Effect |
|---|---|
| `until X` / `through X` | Run stages 1..X; later stages are `not run: out of range` |
| `from X` | Stages before X are `skipped (range)`; their artifacts must already exist (spec for plan, plan artifacts for tasks, `tasks.md` for analyze), else `BLOCKED — range needs <artifact>` |
| `only X` | `from X` and `until X` together |

Inside the range, the Step 5 detector still fires: a clarify interrupt runs
even when the clarify stage itself is out of range, and edits stop at the end
of the range (see Step 5). The clarify stage row runs only when in range.
Analyze runs only when in range and `tasks.md` exists; Step 6 remediations
edit only artifacts of in-range stages, and a finding about an out-of-range
artifact is reported `out of range`. The triage pass is read-only and runs
regardless of range, unless skipped. If analyze does not run, the verdict is
never `READY` (Step 8).

---

## Step 5 — Clarify interrupts and the resume point

Clarify runs at stage 2 **and** whenever the detector fires after a stage:

| Trigger | Example |
|---|---|
| `[NEEDS CLARIFICATION]` left by a stage | Marker from specify |
| Stage output contradicts the spec or a DR | Plan chooses polling; FR-004 requires real-time |
| Stage needs a fact the spec lacks | Tasks cannot size an item — no scale stated |
| Analyze finding that is a spec ambiguity | "fast" with no metric |
| Iterate change makes a spec section stale | Step 7 |

Run clarify scoped to the gap — pass a focus hint as its input, shaped like
triage's (`<slug>, <question> (<source>)`) — and answer through Step 3. Then
pick the resume point:

1. **Trace** the spec change by reference (FR/SC ids) and by substance (the
   behavior's terms) across `plan.md`, `research.md`, `data-model.md`,
   `contracts/`, `quickstart.md`, and `tasks.md`.
2. **Resume at the earliest affected stage** and revise forward through every
   later stage, then analyze. Analyze reports only — re-running it repairs
   nothing.
   Revisions stop at the end of `Range`: later stages are left untouched and
   noted `stale: out of range` in the Pipeline table.
3. Resume at the raising stage only when nothing downstream depends on the
   change.

Each interrupt and its reruns count against `autopilot.clarify_max_interrupts`. A
gap that triggers twice is queued with its rank; when the budget is spent,
remaining gaps are queued.

---

## Step 6 — Analyze and remediate (bounded loop)

Route each analyze finding by its ID and severity:

| Finding | Action |
|---|---|
| Mechanical — terminology drift, duplication, broken reference, task ordering, requirement with no task | Edit when `autopilot.auto_remediate: true`; else report |
| Spec-level ambiguity or underspecified requirement | Step 5 interrupt |
| Needs a decision | Step 3 |
| Follows from an Accepted DR | Edit, citing the DR |
| Depends on a queued decision, an existing Proposed DR, a default, or an unconfirmed supersession | Not applied — `blocked by <id>` |
| Constitution conflict (CRITICAL) | Adjust the artifact when the compliant fix is unambiguous and changes no Accepted DR or stated intent; otherwise `BLOCKED`. Never edit the constitution |
| Platform-scoped | ADR handoff |

Record each applied remediation as finding id → `file:line` → change. Re-run
analyze; stop when no applicable findings remain, `autopilot.analyze_max_passes`
is reached, or only blocked findings remain.

---

## Step 7 — Iterate mode: intake and reconcile

Inputs: the last run in `REPORT` (pending items, assumptions, verdict), a
recap brief in this conversation when present (decided, open, next), and the
Change input.

| Intake | Example | Action |
|---|---|---|
| Decision change | "change DR-004 to event-driven" | Draft the superseding DR via record `supersede` and queue it as a `PD-N`; end the run `BLOCKED — awaiting supersession approval (PD-N)` with every dependent edit unapplied, showing record's Step 4 plan plus the impact summary. A "yes" reply or `approve PD-N` is the confirmation and starts the next run (Approvals row) |
| Approvals | "approve PD-3, PD-5 (PD-5 option B)", "approve DR-012" | Re-check each item first: still DR-worthy (routing reference — not SKIP, not platform, not settled by a later change); the chosen option (recommended or named) has evidence meeting `evidence_rule`, re-verified now; supersedes no Accepted DR, unless the item is a supersession draft whose plan an earlier run showed — then this approval is that confirmation. Pass → record writes a new `Accepted` DR for a `PD-N` (superseding when drafted so), or flips an existing Proposed `DR-NNN` in place (same id, same ledger row) — the user's message is the confirmation — then apply the edits it unblocks. Fail → stays pending, naming the failed check |
| Rejections | "reject PD-4" | Mark it rejected in the report — no write; its blocked edits stay unapplied. An existing DR is rejected through record |
| Approving an open item | "approve the T014 blocker" | Refuse — open items are not decisions; give its route |
| Requirement change | "drop export from MVP" | Scoped clarify with the change as the answer → resume rule (Step 5) |
| None stated | "reconcile for review" | Delta sweep, clarify scan, analyze loop |

Never invent a change. When recap's open items suggest one the user has not
stated, ask once.

**Range preflight.** When `Range` is set, an approval that unblocks edits or
confirms a supersession is checked **before** any write: trace its impact
(below) read-only first. If reconciliation would edit an artifact of an
out-of-range stage, or its analyze check is out of range, write nothing — no
DR, no ledger row, no edit. The item stays pending (`PD-N`, or the existing
Proposed `DR-NNN`) and the run ends `BLOCKED — range too narrow for <PD-N>:
needs <wider range, e.g. through analyze>`, listing the out-of-range
artifacts it would touch. Never apply a partial reconciliation and never
mark one `Reconciled`. A Requirement change follows Step 5's range bound
instead.

### Impact graph and reconciliation

Trace the old decision by **reference** (DR id, slug, FR/SC ids) and by
**substance** (the old behavior's terms and constraints, even when worded
differently). Read every candidate hit and classify it:

| Class | Examples | Action |
|---|---|---|
| Active, requires old behavior | FR text, plan section, task, contract, data-model entry | Edit to the new decision |
| Active, independent | Mention that does not depend on the old behavior | Leave |
| Historical | Old DR file, superseded ledger row, clarification session bullets, prior report runs | Never edit |
| Ambiguous | Wording that might depend on the old behavior | Queue with a recommendation |

Link sparingly, as record does: one inline link in `plan.md` or the relevant
FR/SC — do not stamp every edited artifact. Record the change itself as a new
clarification session bullet. Edit in stage order (spec → plan artifacts →
tasks), then run the analyze loop.

**Pass test:** no active artifact still requires the old behavior.
References to the old DR in historical artifacts are expected.

---

## Step 8 — Triage pass and verdict

### Triage pass (read-only)

Run `__SPECKIT_COMMAND_DELIBERATE_TRIAGE__` with input `<FEATURE_DIR> all
full` (the path relative to the project root) and the Step 0 override — a
subagent needs both. `full` because `compact` caps the board per rank. Run
it in a subagent when the agent supports one; otherwise as an inline,
read-only pass. Its board, categories, and resolution queue are data for
this step: its closing capture offer ends the pass, not this run, and autopilot
never runs its capture routes.

Confirm the pass triaged `FEATURE_DIR`: its load line and cited sources must
point inside it. Otherwise discard the board and treat the pass as not run
(`triage pass not run — feature mismatch`).

Merge the board into the pending list (triage does not read `REPORT`):

| Board item | Action |
|---|---|
| Same source or question as a `PD-N` or assumption | Keep it; take triage's rank when higher, plus its slug and focus hint |
| Existing Proposed or `Confidence: Low` DR, or a Revisit Trigger now true | Pending decision under its own `DR-NNN` — never a new id; approvable through Step 7 |
| Spec gap, open marker, failing checklist gate, blocked task | **Open item** with triage's rank, slug, and route — not approvable; its owning command or a later iterate run resolves it |
| `platform-signal` | Platform handoffs; its stage dependency decides the verdict effect |
| Triage board id (`P0-1`) | Not stored — board ids are per run |

A default that triage ranks above `autopilot.defaultable_max_rank` loses its
default status: keep its `mark_assumed` text in place (never silently
rewrite it), list it as an open item at triage's rank routed to
`__SPECKIT_COMMAND_CLARIFY__`, and mark everything built on it in later
stages `blocked by <gap>` — the verdict treats it as unresolved.

When the pass cannot run or is skipped (`autopilot.triage_pass: false`, or
`skip triage` / `no triage` in this run's input), say `triage pass not run`
with the reason in the verdict line and cap the verdict at
`READY WITH PENDING DECISIONS` — an untriaged tree is never `READY`.

### Verdict

Computed from what pending items block, never from queue size:

| Condition | Verdict | Next |
|---|---|---|
| Any pending P0 (decision or open item); a stage not run because of a P0, an awaited ADR handoff, or an unconfirmed supersession; an unresolved CRITICAL finding; a Step 2 gate | `BLOCKED — <head blocker> (blocks X)` | The head blocker's route with its focus hint |
| No blocker, ≥1 pending P1 (decision, open item, or assumption) | `READY WITH PENDING DECISIONS` | Decisions: `__SPECKIT_COMMAND_DELIBERATE_AUTOPILOT__ approve PD-…` / `reject PD-…`; open items: their routes |
| Only P2 pending, or nothing, and analyze ran | `READY` | `__SPECKIT_COMMAND_IMPLEMENT__` (autopilot never runs it) |
| Only P2 pending, or nothing, but analyze did not run (`Range`) | `READY THROUGH <last stage run>` | `__SPECKIT_COMMAND_DELIBERATE_AUTOPILOT__ from <next stage>`; implement is not offered |

Platform handoffs no stage depends on are listed but do not change the
verdict.

---

## Step 9 — Report

Append one run section to `REPORT` — create the file with a
`# Autopilot report — <feature>` heading when absent; never edit earlier runs —
and emit it in chat according to `Report`:

```markdown
## Run N — YYYY-MM-DD · initial|iterate

**Verdict:** READY | READY THROUGH <stage> | READY WITH PENDING DECISIONS | BLOCKED — <reason> (blocks X)
**Context tier:** Rich|Partial|Sparse — found: … · none found: …

### Pipeline
| Stage | Op | Status | Artifact | Note |
|---|---|---|---|---|
| specify | generate | done | spec.md | |
| clarify #1 | — | done | spec.md | 4 answered (1 assumed), 1 open |
| plan | generate | done | plan.md, data-model.md | |
| clarify #2 | — | done | spec.md | raised by plan; resumed at plan |
| tasks | generate | done | tasks.md | |
| analyze ×2 | — | done | — | 6 findings, 4 remediated, 2 blocked |
| triage | — | done | — | full board: 9 items (1 P0 · 5 P1 · 3 P2), 3 new merged |

### Context pack
- Constraints honored: … (source)
- Restrictions applied: …
- Knowledge reused: … (file:line)

### Decisions written
| DR | Question | Chosen | Evidence | Approval |
|---|---|---|---|---|

### Pending decisions
| Id | Rank | Category | Blocks | Question | Recommended | Alternatives | Evidence | Route |
|---|---|---|---|---|---|---|---|---|

### Open items
| Rank | Category | Blocks | Item | Source | Route |
|---|---|---|---|---|---|

### Assumptions (defaulted)
| Gap | Rank | Default | Effort · Cost · Reversibility | Rejected | Revisit trigger | Tagged at |
|---|---|---|---|---|---|---|

### Remediations
| Finding | File | Change | Status |
|---|---|---|---|

### Platform handoffs
| Signal | Stage dependency | Handoff |
|---|---|---|

### Change and reconciliation (iterate only)
- Change: <old> → <new> (DR-NNN, approval status)
- Impact: 7 edited · 2 independent · 3 historical · 1 queued (table: artifact · location · class · action)
- Reconciled: [x] old DR status points forward · [x] old ledger row kept · [x] new row present · [x] every active hit edited or classified · [x] no unqueued ambiguity · [x] analyze clean for the change

### Next
1. <derived from the verdict>
```

Omit empty sections except Verdict, Pipeline, and Next. Pending and open
rows sort P0 first. Decision ids are `PD-N` (stable across runs) or an
existing `DR-NNN`; Evidence `none` means approval will fail its re-check.
Category uses triage's slugs; Route is the owning command plus a focus hint
in triage's shape (`<slug>[ /<sub-area>], <short question> (<id>, <source>)`).
The Pipeline table lists the triage pass as its own row (board size, new
items merged).

**`Report: brief`** (the chat default) prints this digest instead of the full
section. The file still receives the full run section above, and every
verification below still runs.

```markdown
**Run N · initial|iterate — <VERDICT>** (BLOCKED: reason and what it blocks)
Stages: specify ✓ · clarify ✓ · plan ✓ · tasks ✓ · analyze ✓ (4 remediated, 2 blocked) · triage ✓
DRs written: 2 · Pending decisions: 3 · Open items: 2 (1 P0) · Assumptions: 4
Needs you (P0 first, max 3): PD-3 <question> — recommended <option>
Next: 1. <step from the verdict>
Full report: <REPORT> → Run N
```

A stage that was `not run`, a Step 2 gate, or a failed check is always named
in the digest, never summarized away. `Report: full` prints the whole
section as the file does.

Before reporting: re-read every written file, grep the ledger row for every
DR written, confirm every `mark_assumed` tag in the artifacts appears under
Assumptions, and check the verdict against Step 8.

---

## Relationship to other commands

| Command | Difference |
|---|---|
| `__SPECKIT_COMMAND_DELIBERATE_EXPLORE__` | **Read-only** divergence — autopilot starts where explore ends |
| `__SPECKIT_COMMAND_DELIBERATE_RECORD__` | **Writes** DRs — autopilot calls it: `propose-only` to verify, write modes per approval |
| `__SPECKIT_COMMAND_DELIBERATE_RECAP__` | **Read-only** brief — its decided/open/next feeds iterate mode |
| `__SPECKIT_COMMAND_DELIBERATE_TRIAGE__` | **Read-only** queue — autopilot runs it (`all full`) as a separate pass before the verdict and merges its board; provisional ranks use its signals |
| `__SPECKIT_COMMAND_ANALYZE__` | **Read-only** audit — autopilot applies only the remediations its rules allow |
| `__SPECKIT_COMMAND_IMPLEMENT__` | **Writes code** — never run by autopilot; offered only on `READY` |

**Typical flow:**

```
__SPECKIT_COMMAND_DELIBERATE_EXPLORE__  →  __SPECKIT_COMMAND_DELIBERATE_AUTOPILOT__  →  review report  →  approve / change (autopilot iterate)  →  __SPECKIT_COMMAND_IMPLEMENT__
```

---

## Related

- Autopilot guide: `.specify/extensions/deliberate/docs/autopilot.md` — lifecycle, config keys, report file
- Routing reference: `.specify/extensions/deliberate/docs/routing.md` — DR vs ADR vs SKIP plus the platform firewall
- Ledger reference: `.specify/extensions/deliberate/docs/ledger.md` — row statuses and supersession edits
- Template checklist: `.specify/extensions/deliberate/docs/template-checklist.md` — required DR sections and hard fails
- Capture-routing reference: `.specify/extensions/deliberate/docs/capture-routing.md` — where each insight type is written
