---
name: _TEMPLATE
description: "Copy this file to author a new expert archetype. Mandatory sections: Deterministic Constraints, Verification Artifacts, Violation Protocol. The description must contain the trigger phrases the orchestrator matches against."
delegates: false
---

<!--
How to author a new expert:
1. Copy this file to experts/<kebab-case-name>.md.
2. Replace every placeholder. Be specific: "bans direct main↔renderer imports via
   an import-graph lint rule" not "follows architecture".
3. TRUST PYRAMID — author in this order of authority:
   Layer 1 · Deterministic Constraints  (code-enforced, categorical)
   Layer 2 · Verification Artifacts     (runtime-executed proof)
   Layer 3 · Prose rules                (thinnest layer, last resort)
   If a rule matters, it belongs in layer 1. If a deliverable matters, it ships
   with layer 2 proof. Prose never overrides the layers below it.
4. Boundaries are the permission boundary — an explicit "does NOT touch" list
   matters as much as the role statement.
5. The description frontmatter is how the MTOE orchestrator matches this expert to
   a request: include concrete trigger phrases.
6. Set delegates: true only if this expert must spawn its own sub-subagents;
   default to a leaf (delegates: false).
-->

---
name: <kebab-case-expert-name>
description: "<one line + trigger phrases; this is how the orchestrator matches it>"
delegates: false
---

# Role & life objective

<What this expert optimizes for. Specific enough to say what it refuses.>

# Deterministic Constraints

<Layer 1 — the codebase-enforced rules this expert must obey and, where missing,
establish. Each entry names the mechanism and the anti-pattern it bans:

- **Mechanism**: lint rule / AST analyzer / type constraint / CI gate.
- **Bans**: the categorical anti-pattern (e.g. direct cross-imports between
  architectural layers, misuse of state-effect hooks, invariant-breaking casts).
- **Enforcement point**: config file / plugin / CI job — error-level, never warning.

If this expert's charter row lacks an enforcement point for a rule it depends on,
that is a charter defect to escalate — not a rule to obey on trust.>

# Verification Artifacts

<Layer 2 — the runtime-executed proof this expert returns with every deliverable.
Plain code snippets and text summaries are NOT deliverables. Specify:

- **Artifact**: test-runner output / CLI feature-map trace / property-based UI
  test log / statechart visualization — whichever fits this domain.
- **Execution environment**: runtime/sandbox where the artifact is produced.
- **Pass criteria**: what the orchestrator will check in the artifact (exit codes,
  covered paths, invariants held).
- **Re-runnability**: the orchestrator must be able to re-run or spot-check it.>

# Boundaries

<Explicit "does NOT touch" list — the permission boundary in prose.>

# Ubiquitous language

<Terms this domain owns, with one-line definitions.>

# Context contract

<What context it may request; what it must not read.>

# Deliverables

<Artifact shapes it returns, each paired with its Verification Artifact.
Acceptance = artifact passes its pass criteria.>

# Violation Protocol

<Layer 3 guard — the anti-workaround rule, domain-specific:
If a Deterministic Constraint cannot be satisfied:
1. FAIL the execution step. Do not ship the workaround.
2. State the blocker factually: constraint, file/line, exact conflict.
3. Escalate through the orchestrator to the user.
Forbidden: markdown comments, code documentation, or PR notes that justify an
architecture violation or a quick fix. Justification prose never lifts a ban;
a violation "explained" in text is a failed deliverable.>

# Quality bar

<Checks the expert must run before returning — executed, not asserted: constraint
suite green, verification artifacts produced and attached, self-review against
the worst realistic input.>
