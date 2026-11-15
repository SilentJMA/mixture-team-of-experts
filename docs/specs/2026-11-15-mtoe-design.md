# Mixture Team of Experts (MTOE) — Skill Design

Date: 2026-11-16
Status: **Revision 3 — proposed.** Rev 2 approved 2026-11-15. Rev 3 integrates rev 2 into the body and tightens the operational contract. One semantic change needs re-approval: binary acceptance gates replace the 70/30 weighting (§ Phase 5). The per-team folder replaces the flat charter file (§ Skill structure) — cheap to change now, flagged for awareness. Ticket tracking added (§ Ticket track) — additive, does not change phase logic.

## Goal

**Mixture Team of Experts (MTOE)** — the name nods to Mixture-of-Experts: instead of one dense general agent, a router composes a small team of specialized experts per request. `/mtoe` is a user-invoked skill that analyzes any request and turns it into a **composed team of domain-specific experts** — each a narrowly scoped sub-agent — with a single orchestrator that delegates, coordinates, and integrates their work. Like an MoE router, it can also route top-1: a single expert, or **no team at all** when the request doesn't earn orchestration overhead.

## Motivation and sources

Four sources ground the design; each contributes load-bearing claims, not vibes.

1. **"Agents for DDD – Back and Forth"** — Ricci, Burattini, Ciortea, Castellucci, EMAS 2024, pp. 175–188 ([Springer](https://dl.acm.org/doi/10.1007/978-3-031-71152-7_11), [material repo](https://github.com/Agents-and-DDD/EMAS-2024-paper)).
   - **Bounded contexts model as agents**: each expert owns a domain, speaks a ubiquitous language, and declares what it does *not* handle.
   - **Application layer vs domain layer**: the orchestrator coordinates, sequences, integrates; the experts know their domain deeply and nothing else. They exchange **artifacts**, never raw internal context.

2. **"Domain-Specific AI Agents: Why Composition Beats One Big General Agent"** ([ospeks.com](https://ospeks.com/domain-specific-ai-agents-composition-vs-general-agent/)).
   - One agent wired to every tool fragments attention and fails everywhere at once; composed specialists fail in exactly one place.
   - Four-layer anatomy: role + refusals; a tool/context layer scoped so narrowly that **the scope is the permission boundary**; hooks/state; sandbox. Delegation is just another tool-layer entry — some agents are **leaves**. Narrow agents are cheaper and parallelize cleanly.

3. **[mattpocock/skills](https://github.com/mattpocock/skills)** (structure and invocation conventions).
   - **User-invoked** skills are reachable only by the human typing them (`disable-model-invocation: true`; human-facing description, no trigger list). **Model-invoked** skills carry model-facing trigger phrasing and are auto-reachable. A user-invoked skill may invoke model-invoked ones, never another user-invoked one.
   - Dependencies are expressed as explicit instructions to **call the Skill tool with a named skill**, one skill per call — not cross-folder file links or bare `/name` mentions.
   - Small, composable, hackable markdown; no framework to own you.

4. **[mattpocock/skills — to-tickets](https://github.com/mattpocock/skills/blob/main/skills/engineering/to-tickets/SKILL.md)** (ticket track conventions).
   - Breaks work into **tracer-bullet tickets**, each declaring its **blocking edges** (tickets that must complete first).
   - Local tracker: one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered `01`… in dependency order, with `Status: ready-for-agent` and checkbox acceptance criteria.
   - Real tracker (GitHub, Linear): one issue per ticket, native blocking links where available, `ready-for-agent` triage label.
   - MTOE uses the to-tickets skill as a model-invoked companion to publish the charter's work as tickets, and keeps a local ticket track inside each team folder so every deliverable is logged, traced, and marked like a real team.

### Design principles (referenced below)

- **P1 — Bounded contexts.** Every expert has explicit owns / does-not-own / language / deliverables.
- **P2 — Layering.** The orchestrator coordinates and never does domain work; experts do domain work and never coordinate. Experts never talk to each other; all handoffs are artifacts routed through the orchestrator.
- **P3 — Inverted trust pyramid.** Deterministic code constraints > runtime verification artifacts > prose. Prose is the thinnest layer; justification never substitutes for proof.
- **P4 — Scope is the permission boundary** — prompt-enforced in v1; see § Trust model for what that honestly means.
- **P5 — The user owns the spawn gate.** Nothing spawns and nothing mutates the repo before charter approval.
- **P6 — Plain markdown, relative paths, no framework.** The roster is editable by anyone.
- **P7 — Ticket track.** Every spawn, mutation, check, gate verdict, and waiver is logged against a ticket. Nothing is done without a ticket number; nothing is marked done without evidence.

## Scope

### In scope (v1)

- One user-invoked orchestrator skill `SKILL.md` (`/mtoe`).
- A library of **expert archetypes** (markdown) covering core engineering domains.
- The five-phase loop: Triage → Compose → Charter → Delegate → Integrate, with **wave 0** (baseline + constraint gates) inside Delegate.
- Delegation via the harness's subagent facility, one subagent per expert; **capability preflight** and a disclosed **degraded sequential mode** when subagents are unavailable.
- A **team charter** as the deterministic, user-approved artifact of composition, including a **deterministic-constraints table**, a **verification plan**, and **write ownership**.
- **Ticket track**: a parent ticket plus one ticket per expert deliverable, numbered in dependency order, with blocking edges, status transitions (proposed → in-progress → complete/blocked/rejected), and full traceability to charter rows and verification artifacts.
- **Verification-driven delivery**: every deliverable ships a re-runnable verification manifest; acceptance is a binary gate, not a score.
- Composability: after approval, implementation work may hand off to installed model-invoked skills (`writing-plans`, `subagent-driven-development`, `tdd`, `code-review`, `verification-before-completion`) and to the `to-tickets` companion for ticket publication.
- **Companion routing**: `security-compliance` → Cloudflare `security-audit`; `reverse-engineer` → REA (`reverse-engineer-anything`); `to-tickets` → ticket publication for the whole run.

### Out of scope (v1)

- Enforced tool-level sandboxing or harness-native agent configs (Approaches B/C, later).
- Skill-owned code/scripts that parse requests or generate agent configs. (The skill *runs* the target repo's checks and the companions' validators; it ships no code of its own.)
- Long-term memory or cross-session expert state.
- Supervision beyond **one corrective round per expert**; no automatic re-spawn loops.
- **Recursive composition**: no expert or companion may invoke `/mtoe` (user-invoked skills are unreachable from skills, by convention and by harness policy).

## Skill structure

The repository follows the `skills.sh` layout — the installable unit is `skills/mtoe/`:

```
team-of-experts/                     # repository root (GitHub: mixture-team-of-experts)
├── README.md                        # purpose, install, usage, theory links
├── docs/specs/                      # this document
└── skills/mtoe/                     # the installable skill (one unit)
    ├── SKILL.md                     # user-invoked orchestrator (/mtoe)
    └── experts/                     # archetype library (the editable roster)
        ├── _TEMPLATE.md             # canonical shape; "_" prefix = never loaded as an expert
        ├── product-strategist.md
        ├── backend-architect.md
        ├── frontend-engineer.md
        ├── ai-llm-engineer.md
        ├── devops-sre.md
        ├── security-compliance.md
        ├── reverse-engineer.md
        ├── data-engineer.md
        └── qa-test-engineer.md
```

**Archetypes are internal prompt modules, not separately registered skills.** The orchestrator loads them by reading files; the harness never auto-triggers them. This keeps installation to one unit and makes the roster editable without touching any registry. `SKILL.md` must resolve `experts/` **relative to its own file location** at run time, so global, per-project, and hand-copied installs behave identically.

`SKILL.md` frontmatter follows the user-invoked convention:

```yaml
---
name: mtoe
description: Compose a charter-approved team of domain experts for a request, delegate, and integrate verified deliverables.
disable-model-invocation: true   # Claude Code; Codex equivalent: agents/openai.yaml policy.allow_implicit_invocation: false (optional file)
---
```

### Runtime artifact layout (in the target repo)

Each run gets one folder; the charter, its evidence, and its ticket track live together:

```
docs/teams/<YYYY-MM-DD>-<topic>/
├── charter.md                       # approved charter, with approval record
├── baseline.md                      # wave-0 record of pre-existing check results
├── tickets/
│   ├── 01-parent-request.md         # parent ticket for the whole request
│   ├── 02-backend-architect-oauth.md# one ticket per expert deliverable
│   └── ...
└── artifacts/<expert>/
    ├── verification.json            # the deliverable's verification manifest
    └── <check-id>.log               # full output of each executed check
```

(Rev 2 used a flat `docs/teams/YYYY-MM-DD-<topic>-charter.md`; the folder supersedes it so artifacts and tickets have a home. Any user-named path still wins.)

## Orchestrator flow (SKILL.md)

```mermaid
flowchart LR
    T["1 · Triage"] -->|"no team needed"| S["answer directly / recommend one skill"]
    T --> C["2 · Compose"]
    C --> H["3 · Charter (hard gate)"]
    H -->|"approved"| W0["wave 0: baseline + gates"]
    W0 --> D["4 · Delegate (waves 1..n)"]
    D --> I["5 · Integrate"]
    H -.->|"rejected → revise"| C
    I -.->|"unresolved conflict → decide or escalate"| H
```

### Phase 1 — Triage

Extract: **goal**, **deliverable type** (design/analysis, implementation, review, migration, research, ops change), **risk domains** (auth → security; UI → frontend; schema → backend + data), **constraints** (stack, deadlines, compliance posture). Then:

- **No-team exit.** If the request is single-domain and low-risk, answer directly or recommend one existing skill — and say that's what you're doing. Orchestration overhead must be earned.
- **Deterministic-constraint sweep (read-only).** Inventory the repo's existing gates (lint, type checks, CI), identify the stack's known anti-patterns (e.g. main↔renderer cross-imports, state-effect-hook misuse), and draft error-level rules that ban them. **Propose only** — no repo mutation before approval (P5). Enforcement happens in wave 0.
- **Capability & companion preflight.** Detect: subagent facility present? parallel spawns? session resume? `security-audit` skill installed? REA's MCP server connected? `to-tickets` skill available? Record results; missing companions are resolved *in the charter*, not mid-flight.
- **Clarify.** Up to **three** questions, multiple choice preferred. Never compose a team from a vague request; if still unclear, stop and ask.
- **Open the parent ticket.** Create ticket `01-parent-request.md` in the team folder's `tickets/` (or publish via the `to-tickets` skill if configured). Status: `proposed`. Record its ID in the charter.

### Phase 2 — Compose

Select the **smallest team that covers the risk domains**: typically 2–4, default cap 5 (more requires explicit user approval in the charter). A **team of one is valid**. Never spawn a specialist for a domain the request doesn't touch.

**Integration is the orchestrator's job (P2) — there is no "integrating expert."** When the deliverable spans domains and requires product-level judgment (scope, prioritization, trade-offs), add `product-strategist` as a **coherence reviewer**: it returns a scope/priority artifact, it does not merge deliverables or arbitrate conflicts.

For each selected expert, record its bounded context:

| Field | Meaning |
|---|---|
| **Owns** | Decisions and artifacts this expert is accountable for |
| **Does not own** | Explicit refusals — the boundary (frontend does not design DB schema) |
| **Language** | Ubiquitous terms this domain uses, so the charter speaks one language |
| **Deliverables** | Artifact shapes returned, each paired with its verification artifact |
| **Depends on** | Whose deliverables it consumes |
| **Ticket** | The ticket number tracking this expert's deliverable |

**Compose also drafts the ticket track.** One ticket per expert deliverable, numbered in dependency order (blockers first), each with `Blocked by` listing the tickets it depends on. The parent ticket (`01`) is blocked by nothing; every expert ticket is blocked by the tickets of the experts it depends on (from `Depends on`), plus `01`. Integration work (merge + final verification) is its own ticket, blocked by all expert tickets.

### Phase 3 — Charter (hard gate)

Write `docs/teams/<YYYY-MM-DD>-<topic>/charter.md` (or the user's path):

```md
# Team Charter: <request title>
Status: proposed | approved (<user>, <date>) | superseded
## Goal & deliverables
## Ticket track
| Ticket | Expert / role | Deliverable | Blocked by | Status | Verification manifest |
| 01 | orchestrator | Parent request — charter + team | None | proposed | N/A |
| 02 | backend-architect | OAuth token exchange | 01 | proposed | artifacts/backend-architect/verification.json |
## Roster
| Expert | Bounded context | Owns | Explicitly not | Deliverable → verification artifact | Ticket |
## Interfaces
<producer → consumer, artifact, format>
## Deterministic constraints
| Rule | Anti-pattern banned | Enforcement point (error-level) | Existing/new | Waiver + reason (only if unenforceable) |
## Verification plan
| Check id | Command | Pass criterion (machine-checkable) | Wave | Orchestrator re-runs? |
## Write ownership
| Path/glob | Owner expert | Wave |        ← implementation teams; disjoint within each wave
## Execution plan
wave 0: baseline + gates · wave 1: independent experts · wave 2+: dependents · integrate: final verification
## Environment & capabilities
<subagent mode: parallel | sequential | degraded; companions available + chosen mode; tool commands>
## Integration & acceptance
<merge order; conflict rules; acceptance gates; escalation triggers>
## Approval
<user, date — recorded before anything spawns>
```

**The user must approve the charter before any expert spawns or any file changes.** This is the control point: the user sees the whole team, its boundaries, its constraints, its ticket track, and its proof plan — and can reshape it cheaply. Rejection → revise composition → re-present. Prose-only rules are accepted **only** with a recorded waiver reason in the constraints table (P3).

### Phase 4 — Delegate

**Wave 0 (orchestrator, immediately after approval):**

1. Mark the parent ticket `01` status `in-progress`.
2. Run the repo's existing checks; record results in `baseline.md`. Pre-existing failures are baseline facts, never attributed to experts.
3. Land the charter's new constraint gates.
4. **Self-test each new gate**: cheaply seed the banned pattern, confirm the gate errors, revert. A gate that cannot fail is prose (P3).

**Prompt assembly** — each expert's prompt contains exactly, in order:

1. Its archetype file, verbatim (role, constraints, boundaries, language, quality bar).
2. Its charter slice: roster row, interfaces where it is producer or consumer, constraints table, its verification-plan rows, its write set, its ticket number.
3. The task brief (goal + constraints from triage).
4. Upstream **artifact paths** (wave ≥ 2), not transcripts.
5. Environment facts: repo root, commit, tool commands, artifact output directory.

**Nothing else.** Excluded by rule: the user transcript, other experts' prompts or reasoning, unrelated files. An expert needing more context requests specific files via its context contract; the orchestrator supplies paths or contents.

**Parallelism.** Experts share a wave only if they have no artifact dependency **and disjoint write sets**. Overlapping writers are serialized into later waves. Optionally use per-expert git worktrees where the harness supports them cleanly; merging is the orchestrator's job in Integrate. Writes outside the owned set are boundary violations.

**Leaves by default.** An expert returns deliverables; it spawns sub-subagents only if its archetype declares `delegates: true` with an explicit rule.

**Follow-ups.** At most **one corrective round per expert**: resume via session id where supported; otherwise a single corrective re-spawn carrying the prior deliverable + a delta brief (this counts as the same round). No other loops.

**Ticket transitions during Delegate:** when an expert spawns, mark its ticket `in-progress`. When its deliverable lands with a passing verification manifest, mark the ticket `complete` and update the charter's ticket track. If blocked, mark `blocked` with a factual blocker statement.

**Degraded mode.** If the harness has no subagent facility, run experts **sequentially in-context with a context firewall**: reload the archetype per expert, work only from the charter slice + upstream artifacts, close each expert's manifest before starting the next. Isolation is weaker; the report must disclose it.

**Every deliverable lands with its verification manifest** (below). A deliverable without artifacts is *incomplete*, not "done with caveats."

#### Verification contract

Each expert writes `artifacts/<expert>/verification.json`:

```yaml
expert: backend-architect
deliverable: oauth-token-exchange
ticket: 02
repo_commit: <sha>
status: complete | incomplete | blocked
checks:
  - id: unit-auth
    command: pnpm vitest run src/auth --reporter=json
    cwd: .
    env: { NODE_ENV: test }
    pass_criterion: "exit 0; 0 failed tests"
    exit_code: 0
    log: unit-auth.log
blockers: []          # factual statements only, no justification prose
```

Rules:

- **Re-runnable**: exact command, cwd, env, commit. The orchestrator re-runs every check marked *re-runs* in the verification plan — at least the cheapest deterministic check per deliverable. Non-reproducible pass = fail.
- **Flake policy**: one re-run of a suspicious result; still inconsistent → report as fail. No retry-to-green.
- **Logs are files**; reports carry excerpts + paths. Scrub secrets from logs and manifests.
- Pass criteria are machine-checkable (exit codes, counts, schema validation) — never "looks correct."

### Phase 5 — Integrate

1. **Acceptance gate (binary, per deliverable).** Accepted iff: shape matches the charter spec; all its verification-plan checks pass with artifacts; designated re-runs reproduce; no unwaived constraint violations; writes stayed inside the owned set. Any failure → incomplete → one corrective round.
   *This replaces rev 2's "~70% runtime / ~30% text" weighting: a weighted score lets prose buy back trust, which the inverted pyramid forbids. Text-level interface review remains — as a **veto** on contract coherence. It can reject a deliverable; it can never rescue one.*
2. **Ticket transitions on acceptance.** Accepted → ticket `complete`. Rejected → ticket stays `in-progress` for one corrective round; still failing → `blocked`, escalate to user.
3. **Arbitrate boundary conflicts** per the charter's integration rules (backend wants sync, frontend wants streaming → orchestrator decides). Experts never negotiate directly (P2). Unresolved → escalate to the user with both options and a recommendation.
4. **Final verification**: run the full check suite + all gates at merged HEAD (`verification-before-completion` discipline). Green at merge, not green in isolation.
5. **Report** with full traceability: deliverable → expert → charter row → ticket; artifact paths; orchestrator re-run results; waivers; gaps — including any orchestrator-authored fixes, degraded mode, and missing companions.

## Expert archetype shape

Each file in `experts/` follows the four-layer anatomy with the trust pyramid inverted (P3): deterministic constraints first, runtime proof second, prose last.

```md
---
name: <kebab-case-expert-name>
description: <model-facing one-liner + trigger phrases; how the orchestrator matches it>
delegates: false          # true only with an explicit delegation rule below
companions: []            # e.g. [security-audit] or [rea]; orchestrator preflights these
---
# Role & life objective
<What this expert optimizes for. Specific enough to say what it refuses.>
# Deterministic Constraints        ← MANDATORY (layer 1)
<Lint rules / AST analyzers / type constraints / CI gates — each naming the
anti-pattern it bans and its error-level enforcement point. Never warnings.>
# Verification Artifacts           ← MANDATORY (layer 2)
<Runtime-executed proof per deliverable: test-runner output, CLI feature-map
trace, property-based UI test log, statechart visualization. Pass criteria
and re-run commands stated. Snippets and summaries are not deliverables.>
# Boundaries                       ← explicit "does NOT touch" list
# Ubiquitous language              ← terms this domain owns
# Context contract                 ← what it may request; what it must not read
# Deliverables                     ← each paired with its verification artifact
# Violation Protocol               ← fail the step; state the blocker factually
                                     (rule, location, failing command); escalate.
                                     No justification prose anywhere — code comments,
                                     docs, PR notes. A violation "explained" is a
                                     failed deliverable.
# Quality bar                      ← executed checks, not asserted ones
```

Each expert also knows its ticket contract: it receives its ticket number in the charter slice, writes `ticket: <NN>` in its `verification.json`, and the orchestrator updates the ticket's status when the gate passes or fails.

## Companion skill integrations

Only the orchestrator routes to companions, and only **through the accountable expert**, as an explicit Skill-tool call (one skill per call). Experts may invoke model-invoked companions; never user-invoked skills; never `/mtoe`. Availability is decided at preflight and recorded in the charter with the chosen mode.

### Ticket publication → `to-tickets` (model-invoked companion)

Source: https://github.com/mattpocock/skills/blob/main/skills/engineering/to-tickets/SKILL.md

- **Trigger**: at charter approval, the orchestrator uses `to-tickets` to publish the charter's work as tickets (or writes them locally following the same template if no tracker is configured).
- **Local tracker**: one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered `01`… in dependency order, each with `Status: ready-for-agent` and checkbox acceptance criteria. MTOE additionally mirrors the ticket track inside `docs/teams/<YYYY-MM-DD>-<topic>/tickets/` so the charter and its tickets are co-located.
- **Real tracker**: one issue per ticket in dependency order (blockers first), native blocking links where the platform has them, `ready-for-agent` triage label.
- **Blocking edges**: derived from the charter's `Depends on` / write-ownership — ticket N is blocked by every ticket whose deliverable N consumes.
- The orchestrator never closes or modifies a parent issue; it only publishes child tickets and updates statuses.

### Security audits → Cloudflare `security-audit`

Source: https://github.com/cloudflare/security-audit-skill

- **Trigger**: a charter row for `security-compliance` requires a codebase audit, vulnerability review, or pen test.
- **Routing**: the expert produces its deliverable through the companion's six phases — reconnaissance → coverage-led hunting → candidate validation → structured output → independent record verification → target-neutral reporting — inheriting `findings.json` (`confirmed` / `needs_validation` / `rejected`), validated by the skill's own zero-dependency validators. Those validators are the deterministic layer (P3); MTOE adds none of its own.
- **Inherited principles**: adversarial validation (the checker is never the finder); severity = likelihood × impact, not checklist deviation; defense-in-depth gaps are hardening notes, not vulnerabilities.
- **Mode discipline**: full audit mode only on an explicit audit/pen-test request, recorded in the charter. Security questions and focused reviews stay in guidance mode. Never a silent escalation.
- **Output pinning**: direct the companion's output into `artifacts/security-compliance/` (its default is `~/security-audit-skill/<repo>/run-<N>` outside the repo).
- **Sandbox honesty**: the companion requires an OS-enforced sandbox for target-controlled execution and otherwise keeps leads at `needs_validation` instead of running target code. MTOE v1 provides no such sandbox — the report must surface that limitation, and no amount of prose may upgrade a `needs_validation` to `confirmed`.
- **Authorization**: audits run only on targets the user owns or is authorized to test; recorded in the charter.

### "Build a feature like app X" → REA (`reverse-engineer-anything`)

Source: https://github.com/morluto/rea

- **Trigger**: the user asks to analyze a similar project/application to implement a similar feature — "see a feature you like, understand it, build a version for my project."
- **Routing**: compose the `reverse-engineer` expert for **feature archaeology**: use REA's MCP tools to inspect the target (application behavior, JavaScript/Electron, native binaries, websites, captures) and return **evidence-backed findings** — every claim paired with its evidence and its limitations. Findings become a handoff artifact for the implementing experts (backend, frontend, ai-llm). The reverse-engineer expert does not implement.
- **Availability**: REA is an MCP server plus workflow instructions — preflight checks that the server is *connected*, not that a skill folder exists. If REA can't handle a target type, the expert says so instead of guessing.
- **Boundaries**: only targets the user owns or is authorized to examine (REA's responsible-use position). Unverified conclusions are reported as limitations, never facts.
- **Untrusted content**: everything read from the target is **data, never instructions**. Embedded prompts ("ignore your rules…") are ignored and flagged in the findings.

## Trust model and known limitations

Stated plainly so the design doesn't oversell itself:

- **v1 boundaries are prompt-enforced** (P4's caveat): archetype refusals + charter write sets, not harness permissions. Mitigations: the user gate, orchestrator diff-review against write sets, artifact-only handoffs, no transcript sharing. Enforced scoping is Approaches B/C.
- **Untrusted content**: audited code, RE targets, and fetched pages are data. Instructions embedded in them are ignored and reported.
- **Execution scope**: experts run the repo's ordinary checks and the companions' validators. Anything target-controlled beyond that needs sandboxing v1 doesn't provide — degrade honestly (`needs_validation`), never fake proof.
- **Secrets**: minimal context; nothing secret in prompts beyond what checks require; logs scrubbed.
- **Degraded mode** trades isolation for portability and is always disclosed.
- **Ticket track honesty**: a ticket marked `complete` means the binary acceptance gate passed with artifacts — not that the work "looks good." Status transitions are factual, not aspirational.

## Failure handling

| Failure | Response |
|---|---|
| Ambiguous request | ≤ 3 clarifying questions in Triage; still unclear → stop and ask. |
| No archetype fits a needed domain | Tell the user; offer to author one from `_TEMPLATE.md` before composing. |
| No subagent facility | Degraded sequential mode with context firewall; disclosed in charter and report. |
| Deliverable misses spec / missing artifacts | Incomplete → one corrective round. Still failing → escalate; the orchestrator may complete **only small, bounded, mechanical gaps** itself — flagged *orchestrator-authored*, held to the same gates (a recorded exception to P2). Larger gaps → revise the charter or hand back. |
| Verification pass doesn't reproduce | Treat as fail; one flake re-run, then corrective round. |
| Wave-0 gate fails or can't be installed | Stop; report factually; propose a fix wave or a recorded waiver — user decides. |
| Expert writes outside its owned set | Boundary violation → corrective round; orchestrator reviews the diff before merge. |
| Boundary conflict between deliverables | Orchestrator decides per charter; unresolved → escalate with both options + recommendation. |
| User rejects charter | Revise and re-present; nothing spawns, nothing mutates. |
| Companion unavailable | Preflight catches it; charter records the fallback (expert's own bounded review) and the report states the gap. Runtime disappearance → same fallback, no silent substitution. |
| Context too large for a prompt | Supply paths via the context contract, never dumps; split the deliverable across waves if needed. |
| Ticket publication fails | Write tickets locally in `docs/teams/<topic>/tickets/` following the to-tickets template; report the fallback. |

## Installation and distribution

Modeled on mattpocock/skills and cloudflare/security-audit-skill: no build step, no plugin host — the repo *is* the package. Requires Node.js ≥ 22.20 for the CLI.

```bash
# project-level (from your project directory)
npx skills add https://github.com/<owner>/mixture-team-of-experts --skill mtoe

# user-level, every project
npx skills add <owner>/mixture-team-of-experts --skill mtoe -g

# target a specific agent non-interactively
npx skills add <owner>/mixture-team-of-experts --skill mtoe -a claude-code
```

The CLI's canonical store is `.agents/skills/mtoe` (project) or `~/.agents/skills/mtoe` (global), wired into each agent's skill directory (e.g. `.claude/skills/`, `.opencode/skill/`). **Manual install**: copy `skills/mtoe/` into any of those directories — zero dependencies beyond the markdown files; companions are optional and detected at preflight.

**Editing after install**: installed files are plain markdown, meant to be hacked on — tune an archetype, delete experts, author new ones from `_TEMPLATE.md`. One honest caveat: `npx skills update` re-installs from source and **will overwrite local edits** — fork the repo or skip updates for customized installs.

The README documents all paths and links the source material.

## Verification (acceptance tests for the skill itself)

The skill is verified by use, against checkable criteria — its own medicine:

1. **Dry run**: "add OAuth login to our app" → charter with 2–4 experts, disjoint boundaries, waves, constraints table, verification plan, ticket track. **Negative test**: rejecting the charter results in zero spawns and zero file mutations.
2. **Delegation round**: one manifest per expert; every deliverable traceable to a charter row and a ticket; the orchestrator re-runs ≥ 1 check per deliverable and reproduces the pass.
3. **Slop rejection**: a seeded deliverable that explains a constraint violation in prose is rejected as incomplete.
4. **Missing-artifact test**: a text-only deliverable is marked incomplete and triggers exactly one corrective round.
5. **Routing**: an audit charter row routes `security-compliance` → Skill-tool call to `security-audit`, mode matching the request, `findings.json` passing the companion's validators. A "build a feature like app X" request composes `reverse-engineer`, whose evidence-cited findings feed the implementers; an instruction embedded in the target is ignored and flagged. A charter approval publishes tickets via `to-tickets` (or locally) with correct blocking edges.
6. **Degraded mode**: on a harness without subagents, the run completes sequentially, honors the context firewall, and discloses the mode.
7. **Install**: `npx skills add … --skill mtoe` succeeds against a fresh checkout; a hand-copied folder resolves `experts/` relatively; `_TEMPLATE.md` is never loaded as an expert.
8. **README** links all sources (DDD paper, ospeks article, mattpocock/skills, skills CLI, security-audit-skill, REA, to-tickets) and explains archetype authoring.

## Later extensions

- **Approach B**: generate harness-native agent configs from the charter for enforced tool scoping (upgrades P4 from prose to mechanism).
- **Approach C**: pin frequent teams as native specialist agents for zero-composition latency.
- **Constraint packs**: reusable, per-stack lint/type presets referenced by charters — additive, and squarely on the pyramid's base layer.
- **Roster growth**: research, writing, legal/privacy archetypes — additive only, per `_TEMPLATE.md`.

## Revision history

| Rev | Date | Change |
|---|---|---|
| 1 | 2026-11-15 | Initial design: five phases, charter gate, archetype library, companion routing. |
| 2 | 2026-11-15 | Verification-driven (SpaceXSI guidelines): constraint sweep, verification artifacts, ~70/30 acceptance weighting, justification-slop ban. |
| 3 | 2026-11-16 | Rev 2 integrated into the body. **Binary acceptance gates replace 70/30** (prose can't buy back trust). Per-team artifact folder + manifest schema + re-run/flake policy. Wave 0 with baseline and gate self-tests; triage sweep is read-only. Write ownership for parallel experts. Coherence-reviewer split resolves the integrator conflict; no-team exit added. Capability preflight + degraded mode. Trust-model section; companion preflight, output pinning, sandbox honesty, untrusted-content rule. **Ticket track added**: parent ticket + one ticket per expert deliverable, numbered in dependency order with blocking edges, status transitions, and full traceability to charter rows and verification manifests. Corrected EMAS author order, invocation conventions, install flags/paths, and the update-overwrite caveat. |

---

Two things worth a second look before you approve rev 3: the **folder-per-team layout** breaks rev 2's charter path convention (cheap to change now, annoying later), and the **binary gate** is stricter than 70/30 — teams will report "incomplete" more often, which is the point, but it changes what "done" feels like. If you want a softer landing, the gate could allow charter-recorded, user-approved waivers per check — same mechanism the constraints table already uses. The ticket track is additive: it co-locates with the team folder and follows the to-tickets template, so it can be dropped in without changing any phase logic.