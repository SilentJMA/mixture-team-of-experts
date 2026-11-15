---
name: _TEMPLATE
description: "Copy this file to author a new expert archetype. Keep the six sections; fill every one. The description must contain the trigger phrases the orchestrator matches against."
delegates: false
---

<!--
How to author a new expert:
1. Copy this file to experts/<kebab-case-name>.md.
2. Replace every placeholder. Be specific: "designs REST contracts" not "does backend work".
3. Boundaries are the permission boundary — an explicit "does NOT touch" list matters
   more than the role statement.
4. The description frontmatter is how the MTOE orchestrator matches this expert to a
   request: include concrete trigger phrases.
5. Set delegates: true only if this expert must spawn its own sub-subagents; default
   to a leaf (delegates: false).
-->

---
name: <kebab-case-expert-name>
description: "<one line + trigger phrases; this is how the orchestrator matches it>"
delegates: false
---

# Role & life objective

<What this expert optimizes for. Specific enough to say what it refuses.>

# Boundaries

<Explicit "does NOT touch" list — the permission boundary in prose.>

# Ubiquitous language

<Terms this domain owns, with one-line definitions.>

# Context contract

<What context it may request; what it must not read.>

# Deliverables

<Artifact shapes it returns, with acceptance criteria.>

# Quality bar

<Checks it must run before returning (builds, tests, lint, self-review).>
