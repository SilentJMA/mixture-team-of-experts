---
name: _TEMPLATE
description: "Copy this file to author a new expert archetype. Mandatory sections: Deterministic Constraints, Verification Artifacts, Boundaries, Ubiquitous language, Context contract, Deliverables, Quality bar."
delegates: false
companions: []
---

<!--
How to author a new expert:
1. Copy this file to experts/<kebab-case-name>.md.
2. Replace every placeholder with concrete, domain-specific rules and proof.
3. TRUST PYRAMID — author in this order of authority:
   Layer 1 · Deterministic Constraints  (code-enforced, error-level, categorical)
   Layer 2 · Verification Artifacts     (runtime-executed proof)
   Layer 3 · Prose rules                (thinnest layer, last resort)
4. Boundaries are the permission boundary — state what this expert owns and what
   belongs to sibling bounded contexts.
5. The description frontmatter is how the MTOE orchestrator matches this expert to
   a request: front-load the domain and include distinct trigger terms.
6. Set delegates: true only if this expert must spawn its own sub-subagents;
   default to a leaf (delegates: false).
7. companions: list the model-invoked companion skills or MCP servers this expert
   routes through (e.g. [security-audit], [rea]).
8. model: optional — pin a harness model ID (e.g. providerID/modelID) only when
   this domain has a specific capability or cost profile; omit to let the
   orchestrator right-size from the user's available models at compose time.
9. Shared execution rules (Violation Protocol, verification.json schema, and
   ticket transitions) live in the orchestrator's subagent dispatch envelope in
   SKILL.md — do not duplicate them here unless adding domain-specific escalation
   triggers.
-->

---
name: <kebab-case-expert-name>
description: "<domain summary>. Triggers: <distinct trigger terms>."
delegates: false          # true only with an explicit delegation rule below
companions: []            # e.g. [security-audit]; orchestrator preflights these
model: <optional — harness model ID, e.g. providerID/modelID; omit for right-sizing>
---

# Role & life objective

<What this expert optimizes for, specific enough to define its refusals.>

# Deterministic Constraints

- **<Rule 1 — e.g. Layer import boundary>**: <mechanism: lint rule / AST check / type constraint / CI gate> — bans <categorical anti-pattern> at error severity.
- **<Rule 2 — e.g. Domain invariant type safety>**: <mechanism> — bans <anti-pattern>.
- **Charter gate requirement**: every constraint this domain relies on must appear in the charter's Deterministic Constraints table with an error-level enforcement point or a user-approved waiver. Missing enforcement point → escalate as a charter defect before writing implementation code.

# Verification Artifacts

Every deliverable ships with executed proof recorded in `artifacts/<expert>/verification.json`:

- **<Deliverable 1>**: <test-runner output / CLI trace / benchmark / schema check> executed in <runtime/sandbox>, passing when <machine-checkable pass criterion: exit 0, 0 failures, invariant assertion count>.
- **<Deliverable 2>**: <artifact + command + machine-checkable pass criterion>.

# Boundaries

Owns <explicit domain scope>. Hands <adjacent domain A> to `<expert-a>`, <adjacent domain B> to `<expert-b>`.

# Ubiquitous language

- **<Term 1>** — <one-line definition>.
- **<Term 2>** — <one-line definition>.

# Context contract

Requests only: <specific slices, schemas, contracts, or configs needed>. Leaves unrelated feature internals out of context; asks the orchestrator for file paths or slices when blocked.

# Deliverables

- **<Deliverable 1>** paired with `<verification check id>`.
- **<Deliverable 2>** paired with `<verification check id>`.

# Quality bar

Completion criteria (all executed before returning `status: "complete"`):
- [ ] All charter-assigned deterministic constraint checks exit `0`.
- [ ] Every deliverable's verification command has been executed, logs written to `artifacts/<expert>/`, and `verification.json` populated.
- [ ] <Domain-specific edge-case or adversarial self-check>.
