---
name: mtoe
description: "Mixture Team of Experts — analyze a user request, compose a small team of domain-specific experts (bounded contexts), get charter approval, delegate each expert to its own subagent with narrow context, then integrate deliverables. Use when the user says /mtoe, 'assemble a team of experts', 'put a team on this', 'who should work on this', or wants parallel specialist coverage of a non-trivial request instead of one general-purpose agent."
---

# Mixture Team of Experts (MTOE)

Composition over one general agent. A request becomes a small team of experts; each
expert is a **bounded context** with its own domain, language, and explicit refusals;
one orchestrator (you) coordinates them. Experts are the domain layer — you are the
application layer. They exchange **artifacts**, never raw internal context.

Grounding: *Agents for DDD* (EMAS 2024) — bounded contexts as agents; *Composition
beats one big general agent* (ospeks.com) — the 4-layer anatomy, narrow context as
the permission boundary; mattpocock/skills — user-invoked orchestrator, model-invoked
archetypes; SpaceXSI guidelines — deterministic code constraints and execution-driven
verification artifacts over markdown rules and text handoffs.

## Trust pyramid (inverted)

Rules are enforced in this order of authority, strongest first:

1. **Deterministic code constraints** — lint rules, AST analyzers, type-level bans,
   CI gates that categorically forbid known anti-patterns. These do not depend on
   anyone reading prose.
2. **Runtime verification artifacts** — executed proof: test-runner outputs, CLI
   feature-map traces, property-based UI test logs, statechart visualizations.
3. **Prose rules** — the thinnest top layer, used only for what cannot be enforced
   or executed.

A rule that matters becomes a constraint; a deliverable that matters comes with
execution proof. Justification text never substitutes for either.

## Setup: locating the expert library

Resolve the `experts/` directory **relative to this file's location** — never against
the working directory. If `experts/` is missing or empty, say so and stop; do not
improvise experts from memory unless the user explicitly asks you to author a new
archetype first (see Failure handling).

## Phase 1 — Triage

Extract from the request:

- **Goal** — the outcome the user wants.
- **Deliverable type** — design/analysis, implementation, review, migration, research, ops change.
- **Risk domains** — which bounded contexts the request touches (auth → security; UI → frontend; schema → backend + data; "build X like app Y" → reverse-engineer).
- **Constraints** — stack, deadlines, compliance posture.

If the request is ambiguous, ask the user **up to three** clarifying questions
(multiple choice preferred). Do not compose a team from a vague request.

### Deterministic-constraint sweep (mandatory)

Do not rely on markdown instructions alone for any rule the codebase could enforce.
Inspect the target codebase and establish or enforce **deterministic guardrails**
that categorically ban known anti-patterns — for example: direct cross-imports
between architectural layers (e.g. main ↔ renderer), misuse of state/effect hooks,
or invariant-breaking type looseness. Mechanisms, strongest first:

- **Lint rules** with error-level severity (never warnings for architecture bans).
- **AST analyzers / custom lint plugins** where pattern-matching configs fall short
  (import-graph rules, hook-call-shape checks).
- **Type-level constraints** (branded/opaque types, `tsconfig` strictness, banned
  APIs via module types) that make the anti-pattern unrepresentable.
- **CI gates** that run the above on every change.

Record in the charter: which guardrails already exist, which this effort must add,
and which anti-pattern each one bans. A rule stated only in prose is accepted only
if you record why it cannot be enforced deterministically.

## Phase 2 — Compose

1. Load the archetype `description` frontmatter of every file in `experts/` and match
   archetypes to risk domains.
2. Select the **smallest team** that covers them: **2–5 experts by default**. Never
   spawn a specialist for a domain the request does not touch.
3. For multi-expert teams whose deliverable spans domains, include exactly one
   **integrating expert** (default: `product-strategist`, acting as the lead voice).
4. For each expert record its bounded context: **Owns / Does not own / Language /
   Deliverables / Depends on** (fields are defined in each archetype).

## Phase 3 — Charter (hard gate)

Write the charter to `docs/teams/YYYY-MM-DD-<topic>-charter.md` in the current
working repository by default, or to any path the user names:

```md
# MTOE Team Charter: <request title>
## Roster
| Expert | Bounded context | Owns | Explicitly not | Deliverables |
## Deterministic constraints
<guardrails this effort establishes or enforces: lint rule / AST analyzer / type
constraint / CI gate — each naming the anti-pattern it bans and its enforcement point>
## Interfaces
<who hands what artifact to whom, in what format>
## Execution plan
<wave 1: parallel experts with no dependencies; wave 2: dependents; …>
## Verification plan
<per expert: the runtime artifact that proves its deliverable — test-runner output,
CLI feature-map trace, property-based UI test log, statechart visualization — and
how the orchestrator will check it>
## Integration & verification
<how deliverables merge; which artifacts the orchestrator re-runs before reporting done>
```

**The user must approve the charter before any expert spawns.** Present it, wait for
approval, revise on rejection. Nothing spawns until approved.

## Phase 4 — Delegate

For each expert, spawn one subagent with the harness's subagent facility:

- **Prompt composition** = the expert's archetype file (full text) + that expert's
  charter slice (task brief, owns/does-not-own, deliverable spec) + the **minimal
  shared context** the task requires. Nothing else — narrow context is the point.
- **Parallelism**: experts with no mutual dependencies spawn in the same wave;
  dependent experts spawn after their dependencies' deliverables exist and receive
  **only those deliverables**, not the whole transcript.
- **Leaves by default**: an expert returns deliverables; it does not spawn
  sub-subagents unless its archetype frontmatter says `delegates: true`.
- **Follow-ups**: at most one clarification round per expert, via the subagent
  session id. Then integrate with what you have.

### Verification-first delivery (mandatory)

Delegated subagents do **not** return plain code snippets or text summaries as
final deliverables. Each expert must execute its changes in a runtime/sandbox
environment and return structured **Verification Artifacts**, e.g.:

- test-runner output (exit code, counts, durations) for the seams it touched;
- CLI feature-map traces showing the executed path through the feature;
- property-based UI test logs (generated interaction suites, not one-off scripts);
- statechart visualizations/renderings proving state-machine behavior.

Code without execution proof is an **incomplete deliverable**, not a draft.

### Anti-workaround rule (justification-slop ban)

Experts are strictly forbidden from producing markdown comments, code
documentation, or PR notes that justify architecture violations or quick fixes —
explanatory prose never lifts a ban. If a deterministic constraint cannot be
satisfied:

1. **Fail the execution step** — stop, do not ship the workaround.
2. State the blocker **factually** (constraint, file/line, exact conflict).
3. Escalate through the orchestrator to the user.

A violation accompanied by justification text is treated as a failed deliverable,
not a nuanced one.

## Phase 5 — Integrate

1. Collect deliverables **and their verification artifacts**; re-run cheap artifacts
   yourself (test suites, lint gates) rather than trusting pasted output.
2. **Hardened integration gate — proof first.** Evaluate each deliverable with
   **~70% of the weight on its runtime verification artifacts** (did the suite pass,
   does the trace cover the feature path, is the statechart consistent with the
   implementation). Prose review carries ~30% and covers only interface consistency
   between deliverables.
3. **Resolve boundary conflicts** yourself (e.g. backend wants a synchronous API,
   frontend wants streaming — decide per the charter's integration rules, or
   escalate to the user with both options stated). Experts never negotiate with
   each other directly. A conflict resolved in text still requires the affected
   experts' artifacts to be re-verified after the change.
4. Report with a **deliverable → expert mapping** and an **artifact → claim
   mapping** — what execution proof backs each deliverable — so the user can trace
   every artifact to its accountable expert and its verification.

## Companion-skill routing

Only route to companion skills **through the accountable expert**, and only to
model-invoked skills — never a user-invoked one.

- **Security audits → `security-audit` (Cloudflare)**. When a charter row for
  `security-compliance` requires a codebase audit, vulnerability review, or pen
  test, the expert produces that deliverable through the companion's six-phase
  workflow (recon → coverage-led hunting → candidate validation → structured output
  → independent record verification → target-neutral report). Explicit audit
  requests run full mode; security questions stay in guidance mode. Never silently
  escalate to full audit mode. Source: github.com/cloudflare/security-audit-skill
- **"Build a feature like app X" → REA (`reverse-engineer-anything`)**. When the
  user asks to analyze a similar project/app to implement a similar feature,
  compose the `reverse-engineer` expert: it uses REA's MCP tools for feature
  archaeology and returns evidence-backed findings as a handoff artifact for the
  implementing experts. It does not implement. Only authorized targets; findings
  cite evidence; unknowns are reported as limitations. Source: github.com/morluto/rea

## Failure handling

| Failure | Response |
|---|---|
| Ambiguous request | Up to 3 clarifying questions in Triage; if still unclear, stop and ask. |
| No archetype fits a needed domain | Tell the user; offer to author a new archetype from `experts/_TEMPLATE.md` first. |
| Expert deliverable misses its spec | One corrective follow-up; if still off, do the work yourself and mark the gap in the report. |
| Boundary conflict between deliverables | Decide using the charter; unresolved conflicts escalate to the user with both options stated. |
| User rejects charter | Revise composition and re-present; nothing spawns until approved. |
| Deliverable arrives without a verification artifact | Incomplete; one corrective round demanding execution proof; if still absent, orchestrator runs the verification itself and marks the expert's deliverable as unproven. |
| Constraint violated with justification text | Reject outright. The prose does not lift the ban — the expert must fix or fail the step and escalate; add the violated anti-pattern to the charter's constraint list if unenforced. |
| Sandbox/runtime execution not possible for a target | The expert states this factually; deliverable is graded `unproven` and never presented as verified. |
| Companion skill not installed | The accountable expert falls back to its own bounded review and states the gap in its deliverable; note the missing companion in the report. |

## Anti-patterns

- **Do not** compose a team for a trivial one-file change — just do it; MTOE is for
  non-trivial, multi-domain requests.
- **Do not** give one expert the whole transcript. Each subagent gets its slice only.
- **Do not** let experts talk to each other. All coordination flows through you.
- **Do not** exceed 5 experts without saying why and getting user agreement.
- **Do not** accept prose as proof. A summary is not a test run; a description is
  not a trace.
- **Do not** allow justification slop anywhere in the pipeline: no comment, doc
  string, or PR note exists to excuse a constraint violation. Violations fail and
  escalate — they never get explained into acceptability.
- **Do not** state a rule in prose when a lint rule, AST check, or type constraint
  could enforce it.
