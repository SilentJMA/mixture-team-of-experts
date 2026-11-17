---
name: mtoe
description: "Mixture Team of Experts — decompose a multi-domain request into a charter-approved team of bounded-context expert subagents, enforce deterministic constraints, and integrate runtime-verified deliverables. Triggers: /mtoe, 'assemble a team of experts', 'put a team on this', 'who should work on this', or multi-domain requests needing parallel specialist execution."
---

# Mixture Team of Experts (MTOE)

Composition over one general agent. A request becomes a small team of experts; each expert is a **bounded context** with its own domain, ubiquitous language, and explicit boundaries; one orchestrator (you) coordinates them. Experts are the domain layer — you are the application layer. Experts exchange **artifacts**, never raw conversation context.

## Trust pyramid (inverted)

Enforce rules in this order of authority, strongest first:

1. **Deterministic code constraints** — lint rules, AST analyzers, type-level bans, and CI gates at error severity that categorically block anti-patterns.
2. **Runtime verification artifacts** — executed proof: test-runner logs, CLI traces, schema/migration runs, eval outputs, and `verification.json` manifests.
3. **Prose rules** — the thinnest top layer, reserved only for what cannot be enforced deterministically or executed.

Every rule that matters becomes a code constraint; every deliverable ships with execution proof.

## Setup: expert library and project root

1. **Expert library (`experts/`)**: resolve `experts/` **relative to this `SKILL.md` file's directory** — never against the working directory. Ignore `_TEMPLATE.md` when loading the roster.
2. **Project root (containment)**: resolve `<project-root>` before any write, in precedence order:
   - The enclosing git repository root (`git rev-parse --show-toplevel`).
   - The current working directory, when it is an existing project (contains source files or a manifest like `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`) and is not the home directory (`$HOME`).
   - **No project present** — create a kebab-case directory named from the request topic (e.g. `./python-tetris-neo-colors/`) in the working directory and use it as `<project-root>`.
3. **Write containment**: write every MTOE file — `<project-root>/docs/teams/<YYYY-MM-DD>-<topic>/`, tickets, verification artifacts, and expert deliverables — strictly inside `<project-root>`. State the resolved `<project-root>` (and whether you created it) when presenting the charter.

---

## Phase 1 — Triage (read-only)

Extract from the request:
- **Goal** — the observable outcome the user wants.
- **Deliverable type** — design/analysis, implementation, review/audit, migration, feature archaeology, or ops change.
- **Risk domains** — which bounded contexts the request touches (e.g. auth → `security-compliance` + `backend-architect`; UI → `frontend-engineer`; schema/ETL → `backend-architect` + `data-engineer`; "build X like app Y" → `reverse-engineer`).
- **Constraints** — stack, deadlines, compliance posture.

**Clarify ambiguity**: if the request is underspecified, ask **up to 3** concise clarifying questions (multiple-choice preferred) and wait before composing.

**No-team exit**: if the request is single-domain and low-risk (e.g. a single-file fix or direct question), state that orchestration overhead is unneeded and handle it directly (or run a single specialist skill).

**Deterministic-constraint sweep (read-only)**:
- Inspect the repo's existing lint, typecheck, AST, and CI gates.
- Identify error-level guardrails that already exist and new ones this effort must add in Wave 0 (e.g. import-boundary rules, strict schema validation, hook/a11y rules).
- Record any rule that cannot be enforced deterministically along with a required waiver reason for the charter.

**Capability & companion preflight**:
- Check subagent support (parallel spawns, session resume, `model` parameter).
- Check model discovery (whether the harness exposes a `models` lookup tool or CLI).
- Check companions required by candidate experts: `to-tickets` skill, `security-audit` skill (for `security-compliance`), `rea` MCP server connection (for `reverse-engineer`).

**Completion criterion**: Goal, risk domains, constraint sweep, and capability preflight are complete, and zero repository files have been mutated.

---

## Phase 2 — Compose

1. **Match archetypes**: read the frontmatter (`name`, `description`, `delegates`, `companions`, `model`) of each file in `experts/` and map experts to the request's risk domains.
2. **Author missing archetypes on demand**: if the request touches a bounded context that no file in `experts/` covers:
   - Propose the new expert archetype (`experts/<kebab-case-name>.md`) to the user, naming its bounded context, deterministic constraints, verification artifacts, and boundaries.
   - Upon user approval (either immediately or at the Charter gate), author `experts/<kebab-case-name>.md` from `experts/_TEMPLATE.md` with concrete domain rules (zero `<Layer ...>` placeholders).
3. **Keep the team small**: select **2–4 experts by default** (cap at 5 unless the user explicitly approves a larger team; a team of 1 is valid). Only include experts whose domain is directly touched.
4. **Coherence reviewer**: when a multi-expert team spans domains with scope or priority trade-offs, include `product-strategist` to produce a scope/acceptance-criteria coherence matrix. Integration and conflict arbitration always stay with you (the orchestrator).
5. **Define bounded contexts**: for each selected expert, define:
   - **Owns** — decisions and files/artifacts this expert is accountable for.
   - **Explicitly not** — adjacent concerns handled by sibling experts.
   - **Language** — domain terms shared across the charter.
   - **Deliverables → Verification** — each output artifact paired with its machine-checkable verification command.
   - **Depends on** — upstream expert artifacts consumed.
   - **Ticket** — ticket number (`02`, `03`, …) in dependency order (`01` is the parent request ticket; integration gets the final ticket blocked by all expert tickets).
6. **Resolve per-expert models** in precedence order:
   1. **User preference** — model the user requested for this expert.
   2. **Archetype default** — `model:` frontmatter in the expert's `.md` file, kept as-is.
   3. **Right-sized recommendation** — when the harness lists available models, pick the **smallest available model** from that list that fits the expert's task tier, plus 1–2 available alternatives:
      - *Smallest/free tier*: docs, test scaffolding, runbooks, mechanical migrations, feature-archaeology notes.
      - *Mid tier*: standard implementation against defined contracts and schemas.
      - *Strongest available tier*: architecture & invariants (`backend-architect`), cross-domain coherence (`product-strategist`), complex security/AI trade-offs.
   4. **`inherited`** — when no model-listing facility exists and none of the above apply, resolve to `inherited` (spawns on the harness default).

**Completion criterion**: Every risk domain is covered by an expert with disjoint write ownership per wave, a resolved model, and a dependency-ordered ticket track.

---

## Phase 3 — Charter (hard gate)

Write the charter to `<project-root>/docs/teams/<YYYY-MM-DD>-<topic>/charter.md` (or a user-specified path) and draft the initial ticket files under `<project-root>/docs/teams/<YYYY-MM-DD>-<topic>/tickets/` (status: `proposed`):

```markdown
# Team Charter: <request title>
Status: proposed | approved (<user>, <date>) | superseded
Project root: <project-root>

## Goal & deliverables
## Ticket track
| Ticket | Expert / role | Deliverable | Blocked by | Status | Verification manifest |
|---|---|---|---|---|---|
| 01 | orchestrator | Parent request — charter & integration | None | proposed | N/A |
| 02 | backend-architect | OAuth token exchange | 01 | proposed | artifacts/backend-architect/verification.json |

## Roster
| Expert | Bounded context | Owns | Explicitly not | Deliverable → verification artifact | Ticket | Model |
|---|---|---|---|---|---|---|

## Interfaces
<producer → consumer, artifact path, schema/format>

## Deterministic constraints
| Rule | Anti-pattern banned | Enforcement point (error-level) | Existing/new | Waiver + reason (only if unenforceable) |
|---|---|---|---|---|

## Verification plan
| Check id | Command | Pass criterion (machine-checkable) | Wave | Orchestrator re-runs? |
|---|---|---|---|---|

## Write ownership
| Path / glob | Owner expert | Wave |
|---|---|---|

## Execution plan
wave 0: baseline + gates · wave 1: independent experts · wave 2+: dependents · integrate: final verification

## Environment & capabilities
<subagent mode: parallel | sequential | degraded; companions available + mode; tool commands>

## Integration & acceptance
<merge order; conflict arbitration rules; binary acceptance gate>

## Approval
<user, date — recorded before Wave 0 or any expert spawn>
```

**Hard gate**: Present the charter and ask the user to approve the **roster (including any newly proposed expert archetypes), boundaries, constraints, ticket track, and model assignments**. Wait for explicit approval before mutating any project code or spawning any subagent. If rejected, revise and re-present.

**Ticket publication**: once approved, update `charter.md` to `Status: approved (<user>, <date>)`. If the `to-tickets` companion skill is installed and a tracker is configured, invoke `to-tickets` to publish the ticket track; otherwise maintain the local ticket files in `<project-root>/docs/teams/<YYYY-MM-DD>-<topic>/tickets/<NN>-<slug>.md`.

---

## Phase 4 — Delegate

### Wave 0 (orchestrator, immediately after charter approval)

1. Mark parent ticket `01` `in-progress`.
2. Run the project's existing test/lint/typecheck suite and record the results in `<team-dir>/baseline.md`. Pre-existing failures are baseline facts, never blamed on experts.
3. Install any `new` error-level constraint gates listed in the charter's Deterministic Constraints table.
4. **Self-test every new gate**: seed a minimal violation, confirm the check exits non-zero, and revert the seed. If a gate cannot be made to fail on a violation, stop and escalate to the user for a fix or charter waiver.

### Waves 1..N (expert subagent dispatch)

**Wave scheduling**: experts run in the same wave only when they have **zero artifact dependencies on each other** and **strictly disjoint write globs**. Serialize overlapping writers into later waves.

**Model parameter**: pass each expert's resolved model via the harness subagent call (e.g. `model: "providerID/modelID"`). Omit the `model` parameter when resolved to `inherited`. If a specified model fails to spawn (unavailable provider or ID), re-spawn immediately on the harness default and log `requested → actual` for the final report.

**Subagent prompt envelope**: assemble each expert's prompt using **only** the following template (never include the raw user conversation transcript, other experts' reasoning, or unrelated files):

````markdown
# Expert Assignment — Ticket <NN>: <deliverable title>

## 1. Your Expert Archetype
<verbatim contents of skills/mtoe/experts/<expert>.md>

## 2. Your Charter Slice
- **Ticket**: <NN>
- **Roster row**: <expert's row from Roster table>
- **Write ownership (strict boundary)**: <globs from Write ownership table for this expert>
- **Interfaces**: <producer/consumer rows involving this expert>
- **Deterministic constraints**:
<Deterministic constraints table from charter>
- **Verification checks assigned to you**:
<Verification plan rows for this expert>

## 3. Task Brief
- **Goal**: <goal from charter>
- **Constraints**: <constraints from charter>
- **Upstream artifacts (read from disk)**: <paths to upstream artifacts, or "None (Wave 1)">

## 4. Environment & Output Paths
- **Project root**: `<project-root>`
- **Team directory**: `<project-root>/docs/teams/<YYYY-MM-DD>-<topic>`
- **Artifact output directory**: `<project-root>/docs/teams/<YYYY-MM-DD>-<topic>/artifacts/<expert>/`
- **Repository HEAD commit**: `<git-sha>`

## 5. Mandatory Execution, Violation & Verification Contract
1. **Stay inside your write set**: modify only files matching your assigned write globs plus your artifact directory (`artifacts/<expert>/`). Need additional read context? Request specific file paths mapped in your Context Contract.
2. **Companion routing**: if your archetype lists a companion in `companions:` and the charter enables it, invoke that model-invoked companion skill/MCP tool directly and pin all outputs into `artifacts/<expert>/`. Never invoke `/mtoe` or spawn subagents unless `delegates: true`.
3. **Violation Protocol (zero justification slop)**: if a Deterministic Constraint cannot be satisfied or blocks your task:
   - **FAIL** the step (`status: "blocked"`). Do not ship a workaround, `eslint-disable`, `@ts-ignore`, or architecture bypass.
   - Record the exact blocker in `verification.json` (`constraint`, `file:line`, `exact conflict`).
   - Never write code comments, docs, or notes excusing a constraint violation.
4. **Verification Manifest**: before returning, execute all assigned verification checks, save their raw logs in `artifacts/<expert>/`, and write `artifacts/<expert>/verification.json` with this exact JSON structure:

```json
{
  "expert": "<expert-name>",
  "deliverable": "<deliverable-slug>",
  "ticket": "<NN>",
  "repo_commit": "<git-sha>",
  "status": "complete",
  "checks": [
    {
      "id": "<check-id>",
      "command": "<exact command executed>",
      "cwd": ".",
      "env": {},
      "pass_criterion": "<machine-checkable criterion>",
      "exit_code": 0,
      "log": "<check-id>.log"
    }
  ],
  "blockers": []
}
```
Scrub any secrets or tokens from logs and `verification.json`.
````

**Degraded sequential mode**: if the harness lacks a subagent tool, execute experts sequentially in-context with a strict context firewall — load one archetype and charter slice at a time, write its deliverable and `verification.json`, and complete its gate check before loading the next expert. Disclose degraded mode in the charter and final report.

**Completion criterion**: Every wave has run in dependency order, ticket statuses reflect current execution state, and every expert has written its deliverables and `artifacts/<expert>/verification.json`.

---

## Phase 5 — Integrate

1. **Binary acceptance gate (per deliverable)**: evaluate each expert's output against five mandatory conditions:
   - Deliverable shape matches the charter specification and interface contracts.
   - `artifacts/<expert>/verification.json` is present, valid JSON, and reports `status: "complete"` with `exit_code: 0` on all assigned checks.
   - **Orchestrator re-run**: re-execute every check marked `Orchestrator re-runs? = yes` in the charter (at least one deterministic check per deliverable) and confirm it reproduces `exit_code: 0`. (If a check is flaky, allow at most one re-run; if still inconsistent, mark failed.)
   - Zero unwaived deterministic constraint violations and zero justification comments/workarounds (`@ts-ignore`, `eslint-disable`, etc.).
   - `git diff` confirms all modified files fall strictly inside the expert's charter write ownership globs.
2. **One corrective round maximum**: if any condition fails, keep the ticket `in-progress` and run at most **one** corrective round (resume the expert's subagent session with the exact failure output, or spawn one corrective subagent with the prior deliverable + delta brief). If still failing after one corrective round:
   - Mark the ticket `blocked`.
   - You (the orchestrator) may patch **only small, mechanical integration gaps** directly — flag them `orchestrator-authored` in the report and subject them to the same verification gates. Escalate larger gaps to the user.
3. **Arbitrate boundary conflicts**: if two deliverables clash at an interface boundary, decide using the charter's integration rules and coherence matrix (or escalate to the user with both options and a recommendation).
4. **Final merged verification**: run the full repository check suite and all Wave-0 constraint gates at merged `HEAD`. Mark accepted expert tickets and the parent/integration tickets `complete`.
5. **Traceability report**: present the final summary containing:
   - Deliverable → Expert → Ticket → Status → Model (`requested → actual` if substituted)
   - Verification manifest & log paths + orchestrator re-run results
   - Constraint gates added and verified in Wave 0 (plus any approved waivers)
   - Any `orchestrator-authored` fixes, missing companions, or degraded mode disclosures

---

## Failure handling reference

| Condition | Action |
|---|---|
| Ambiguous request | Ask ≤ 3 clarifying questions in Triage; wait for answers before composing. |
| Missing domain archetype | Propose a new expert archetype, get user approval, and create `experts/<name>.md` from `_TEMPLATE.md` with concrete Layer 1/2 rules. |
| Invoked outside a project (e.g. `$HOME`) | Create `<kebab-case-topic>/` directory, resolve it as `<project-root>`, and keep all writes inside it. |
| User rejects charter | Revise composition/charter and re-present; zero repo mutations or spawns until approved. |
| Wave-0 gate fails self-test | Stop before Wave 1; report factually and ask user to approve a fix or a recorded charter waiver. |
| Deliverable fails binary gate | Run at most 1 corrective round; if still failing, mark ticket `blocked` and escalate. |
| Expert writes outside owned globs | Revert unauthorized paths and issue 1 corrective round; review diff before merge. |
| Companion skill/MCP unavailable | Detect at preflight, record fallback in charter (`security-compliance` or `reverse-engineer` runs bounded local review; local tickets replace `to-tickets`), and disclose in report. |
| Assigned model fails at spawn | Immediately fall back to harness default model; record `requested → actual` in final report. |
