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
archetypes.

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
## Interfaces
<who hands what artifact to whom, in what format>
## Execution plan
<wave 1: parallel experts with no dependencies; wave 2: dependents; …>
## Integration & verification
<how deliverables merge; what checks run before reporting done>
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

## Phase 5 — Integrate

1. Collect deliverables; check each against its charter spec.
2. **Resolve boundary conflicts yourself** (e.g. backend wants a synchronous API,
   frontend wants streaming — decide per the charter's integration rules, or
   escalate to the user with both options stated). Experts never negotiate with
   each other directly.
3. Run verification (builds, tests, checks as applicable; evidence before "done").
4. Report with a **deliverable → expert mapping** so every artifact traces to its
   accountable expert.

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
| Companion skill not installed | The accountable expert falls back to its own bounded review and states the gap in its deliverable; note the missing companion in the report. |

## Anti-patterns

- **Do not** compose a team for a trivial one-file change — just do it; MTOE is for
  non-trivial, multi-domain requests.
- **Do not** give one expert the whole transcript. Each subagent gets its slice only.
- **Do not** let experts talk to each other. All coordination flows through you.
- **Do not** exceed 5 experts without saying why and getting user agreement.
