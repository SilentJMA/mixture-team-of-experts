# Per-Expert Model Assignment — MTOE Skill Design (Revision 3)

Date: 2026-11-16
Status: Approved for implementation
Parent design: [`2026-11-15-mtoe-design.md`](2026-11-15-mtoe-design.md)

## Goal

Every expert in a composed MTOE team can run on a **specific model**, chosen per role
rather than globally. The orchestrator discovers the models available in the user's
setup, **recommends the smallest model that can credibly do each bounded context's
task** (not the frontier model by default), and the user confirms or edits the
assignments at the existing charter hard gate.

## Motivation and source

1. **AutoGen** (*AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent
   Conversation*, Wu et al., arXiv:2310.01905 — https://arxiv.org/pdf/2310.01905).
   Every AutoGen agent is constructed with its **own `llm_config`** — model choice is
   a per-agent property, and one conversation can mix heterogeneous models. The MTOE
   analog: the model is part of each expert's definition, resolved
   archetype → charter → spawn, exactly as the archetype → charter → prompt
   composition already works.
2. **Existing grounding, extended.** The ospeks source already states that narrow
   agents are cheaper; right-sizing the model per bounded context completes that
   argument. Narrow context is the point of MTOE — most bounded contexts do not need
   a frontier model, and paying frontier cost for well-scoped, well-specified work is
   waste.

## Scope

### In scope

- An **optional `model:` field** in expert archetype frontmatter, documented in
  `_TEMPLATE.md`. Mechanism only — the nine shipped archetypes get **no baked model
  IDs** (they are provider-specific and go stale).
- **Model discovery**: when the harness exposes a model-listing facility (e.g. a
  `models` tool, `opencode models`), the orchestrator enumerates what is actually
  connected and bases recommendations on it.
- **Right-sizing recommendations** per expert, with a one-line rationale.
- **Charter integration**: the roster gains a `Model` column; model assignments are
  visible and editable at the existing approval gate — no new interruption.
- **Delegation**: each subagent spawn receives its resolved model via the harness's
  native model parameter.
- **Fallback and substitution reporting** when an assigned model is unavailable.
- Documentation updates: `README.md` (AutoGen as design source #4, one-line feature
  mention) and a Revision 3 pointer in the parent design doc.

### Out of scope

- A tier-map config layer (`reasoning-heavy` → model mappings).
- Concrete model IDs baked into shipped archetypes.
- Harness-native agent-config generation (parent design, "Later extensions").
- Cost accounting or token budgets per expert.

## Design

### 1. Archetype frontmatter (`experts/_TEMPLATE.md`)

```yaml
---
name: <kebab-case-expert-name>
description: "<one line + trigger phrases; this is how the orchestrator matches it>"
delegates: false
model: <optional — harness model ID, e.g. providerID/modelID; omit for harness default>
---
```

Authoring notes gain one rule: set `model:` when the domain has a specific capability
or cost profile (e.g. a leaf archetype that only drafts test scaffolding may pin a
small model); omit it and let the orchestrator right-size at compose time. Shipped
archetypes are unchanged — no `model:` field, harness default applies.

### 2. Compose (Phase 2) — resolution order, discovery, right-sizing

For each selected expert, resolve its model in this precedence:

1. **User preference** — a model the user named during triage for this role.
2. **Archetype default** — the archetype's `model:` frontmatter, shown as-is
   (no re-recommendation; the archetype author's choice stands, and the user can
   still edit it at the gate).
3. **Right-sized recommendation** — when 1 and 2 are absent and discovery is
   available, the orchestrator recommends the **smallest available model that can
   credibly do the bounded context's task**, with a one-line rationale.
4. **Inherited** — when none of the above apply, the expert spawns on the harness
   default model and the roster says `inherited`.

**Discovery.** If the harness provides a model-listing facility, enumerate the
connected models and recommend only from that list. If it does not, skip discovery
silently — never fabricate a model list.

**Right-sizing guidance** (prose layer — model choice cannot be enforced
deterministically):

| Bounded-context demands | Recommendation tier |
|---|---|
| Narrow, well-specified work: docs, test scaffolding, runbooks, feature-archaeology notes, mechanical migrations | smallest/free tier (e.g. 4B–9B) |
| Standard implementation against already-defined contracts and interfaces | mid tier (e.g. 9B–38B, cheap hosted) |
| Architecture and invariants, cross-domain integration lead, ambiguous trade-offs | strongest available |

The integrating expert (default `product-strategist`) usually merits the strongest
available model; leaf experts executing well-specified slices usually do not.

### 3. Charter (Phase 3) — Model column at the hard gate

The roster table becomes:

```md
| Expert | Bounded context | Owns | Explicitly not | Deliverables | Model |
```

The `Model` cell holds the resolved assignment (`inherited` when none), and for
recommendations one or two **available** alternatives. The gate presentation invites
model edits explicitly, e.g.:

> Approve the roster, boundaries, and model assignments — or name changes?

Because discovery is grounded in the user's actual setup, every name in the column is
spawnable as shown. Editing the charter before approval is the override path; nothing
spawns until approved (unchanged).

**No-discovery path.** If the harness has no model-listing facility and the user gave
no preference, the orchestrator may spend one of its three triage questions on model
or cost preference; otherwise all experts resolve to `inherited` and the charter notes
that recommendations were unavailable.

### 4. Delegate (Phase 4) — spawn with the model

Each subagent spawn passes the expert's resolved model via the harness's native model
parameter (in OpenCode, `model: "providerID/modelID"`). Experts resolved to
`inherited` spawn **without** the parameter, so the harness default applies unchanged.
The charter slice each expert receives is unchanged — the model is spawn plumbing, not
prompt content.

### 5. Integrate (Phase 5) — substitution transparency

The final report's deliverable → expert mapping also records the model each expert ran
on. When a substitution occurred, the report shows **requested → actual** per expert,
so the trace stays honest without blocking the run.

### 6. Failure handling (new rows)

| Failure | Response |
|---|---|
| Assigned model unavailable at spawn (provider not connected, bad ID) | Spawn on the harness default model and record the substitution; never block the run. Report shows requested → actual. |
| No model-listing facility in the harness | Skip recommendations; resolve to archetype defaults / `inherited`, note the gap in the charter, and let the user name models manually at the gate. |

## Example roster row

```md
| Expert | Bounded context | Owns | Explicitly not | Deliverables | Model |
|---|---|---|---|---|---|
| backend-architect | API contracts, schema, invariants | ... | ... | contract, migrations | `acme/72b-reasoning` — invariant-heavy architecture work; alt: `acme/frontier` |
| qa-test-engineer | Test strategy, fixtures | ... | ... | suite + runner output | `acme/4b-free` — well-specified scaffolding; alt: `acme/9b-free` |
| product-strategist | Integration lead | ... | ... | integration plan | `acme/frontier` — cross-domain trade-offs |
```

## Verification

The skill is markdown and is verified by use:

1. **Dry run** — a sample request produces a charter whose roster has a `Model`
   column; each expert carries an assignment or `inherited`, and recommendations are
   sized to the bounded context with a rationale.
2. **Discovery** — with a harness that lists models, every recommended name exists in
   the user's setup; with a harness that does not, no model names are fabricated.
3. **Delegation** — spawns carry the resolved model parameter; `inherited` experts
   spawn without it.
4. **Substitution** — an intentionally unavailable assignment yields a harness-default
   spawn and a requested → actual line in the report.
5. **Docs** — `README.md` lists AutoGen (arXiv:2310.01905) as source #4 with a
   one-line feature mention; the parent design doc carries a Revision 3 pointer to
   this spec.
