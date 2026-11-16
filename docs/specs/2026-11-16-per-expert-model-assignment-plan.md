# Per-Expert Model Assignment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Every MTOE expert can run on a specific model — resolved from archetype frontmatter / user preference / right-sized recommendation — visible in the charter, passed at subagent spawn, with fallback-and-substitution reporting.

**Architecture:** Markdown-only skill, no code. The feature threads through the orchestrator's five phases (`skills/mtoe/SKILL.md`): Compose resolves each expert's model, Charter shows it in a new roster `Model` column at the existing approval gate, Delegate passes it to the subagent spawn, Integrate reports substitutions. The archetype template documents the optional `model:` frontmatter; README and the parent design doc get the AutoGen (arXiv:2310.01905) grounding and revision pointer.

**Tech Stack:** Plain markdown files, git. No build system, no test runner — verification is grep/read-based text checks (TDD does not apply; there is nothing to execute).

**Spec:** [`2026-11-16-per-expert-model-assignment-design.md`](2026-11-16-per-expert-model-assignment-design.md)

---

## File structure

| File | Responsibility | Change |
|---|---|---|
| `skills/mtoe/experts/_TEMPLATE.md` | Canonical shape for authoring new experts | Document optional `model:` frontmatter + authoring rule |
| `skills/mtoe/SKILL.md` | The orchestrator prompt (five phases) | Model resolution + discovery + right-sizing (Phase 2), `Model` roster column + gate wording (Phase 3), spawn-with-model bullet (Phase 4), requested→actual reporting (Phase 5), two failure-handling rows |
| `README.md` | Repo front page | AutoGen as design source #4, one-paragraph feature mention |
| `docs/specs/2026-11-15-mtoe-design.md` | Parent design doc | Revision 3 pointer section, roster-table and failure-row sync |

The nine shipped expert archetypes (`backend-architect.md`, etc.) are **not touched** — per the spec, no baked model IDs.

**How to apply edits:** each edit step gives exact `OLD` and `NEW` text. Apply with a find-and-replace edit tool using `OLD` as the search string (must match exactly once). Do not reflow surrounding prose. Em dashes (—), en dashes (–), and arrows (→) are literal characters — copy them exactly.

---

### Task 1: `model:` frontmatter in the expert template

**Files:**
- Modify: `skills/mtoe/experts/_TEMPLATE.md`

- [ ] **Step 1: Add authoring rule 7 to the comment block**

OLD (lines 22–23):

```
6. Set delegates: true only if this expert must spawn its own sub-subagents;
   default to a leaf (delegates: false).
```

NEW:

```
6. Set delegates: true only if this expert must spawn its own sub-subagents;
   default to a leaf (delegates: false).
7. `model:` (optional) — pin a harness model ID (e.g. `providerID/modelID`) only
   when this domain has a specific capability or cost profile; omit it to let the
   orchestrator right-size from the user's available models at compose time.
```

- [ ] **Step 2: Add the field to the placeholder frontmatter**

OLD (lines 27–30 — the second frontmatter block, not the file's own `name: _TEMPLATE` block):

```
---
name: <kebab-case-expert-name>
description: "<one line + trigger phrases; this is how the orchestrator matches it>"
delegates: false
---
```

NEW:

```
---
name: <kebab-case-expert-name>
description: "<one line + trigger phrases; this is how the orchestrator matches it>"
delegates: false
model: <optional — harness model ID, e.g. providerID/modelID; omit for harness default>
---
```

- [ ] **Step 3: Verify**

Run: `grep -c "model:" skills/mtoe/experts/_TEMPLATE.md`
Expected: `2` (the comment rule and the frontmatter line)

- [ ] **Step 4: Commit**

```bash
git add skills/mtoe/experts/_TEMPLATE.md
git commit -m "docs: document optional per-expert model frontmatter in archetype template"
```

---

### Task 2: SKILL.md Phase 2 — model resolution, discovery, right-sizing

**Files:**
- Modify: `skills/mtoe/SKILL.md` (Phase 2 — Compose)

- [ ] **Step 1: Add resolution step 5 to the Compose numbered list**

OLD (lines 80–81):

```
4. For each expert record its bounded context: **Owns / Does not own / Language /
   Deliverables / Depends on** (fields are defined in each archetype).
```

NEW (append step 5 after it):

```
4. For each expert record its bounded context: **Owns / Does not own / Language /
   Deliverables / Depends on** (fields are defined in each archetype).
5. Resolve each expert's model, in precedence order:
   - **User preference** — a model the user named during Triage for this role.
   - **Archetype default** — the archetype's optional `model:` frontmatter; show it
     as-is and never re-recommend (the archetype author's choice stands).
   - **Right-sized recommendation** — when neither of the above applies, recommend
     the **smallest available model that can credibly do the bounded context's
     task**, with a one-line rationale (see Model discovery and right-sizing).
   - **Inherited** — when none of the above apply, the expert spawns on the harness
     default model and the roster says `inherited`.

### Model discovery and right-sizing

If the harness exposes a model-listing facility (e.g. a `models` tool or an
`opencode models` CLI), enumerate the connected models and recommend **only from
that list** — never fabricate a model name. If it does not, skip discovery silently
and resolve to archetype defaults / `inherited`, noting the gap in the charter.

Right-sizing guidance (model choice cannot be enforced deterministically — prose
layer, thinnest by the trust pyramid):

| Bounded-context demands | Recommendation tier |
|---|---|
| Narrow, well-specified work: docs, test scaffolding, runbooks, feature-archaeology notes, mechanical migrations | smallest/free tier (e.g. 4B–9B) |
| Standard implementation against already-defined contracts and interfaces | mid tier (e.g. 9B–38B, cheap hosted) |
| Architecture and invariants, cross-domain integration lead, ambiguous trade-offs | strongest available |

The integrating expert usually merits the strongest available model; leaf experts
executing well-specified slices usually do not. Model choice is a per-agent
property, not a global one (AutoGen, arXiv:2310.01905).
```

- [ ] **Step 2: Verify**

Run: `grep -n "Model discovery and right-sizing" skills/mtoe/SKILL.md`
Expected: exactly one hit, the `### Model discovery and right-sizing` heading, between Phase 2's list and `## Phase 3 — Charter`

- [ ] **Step 3: Verify right-sizing table is intact**

Run: `grep -c "smallest/free tier (e.g. 4B–9B)" skills/mtoe/SKILL.md`
Expected: `1`

- [ ] **Step 4: Commit**

```bash
git add skills/mtoe/SKILL.md
git commit -m "docs: compose-phase model resolution, discovery, and right-sizing"
```

---

### Task 3: SKILL.md Phase 3 — Model column and gate wording

**Files:**
- Modify: `skills/mtoe/SKILL.md` (Phase 3 — Charter)

- [ ] **Step 1: Add `Model` to the charter roster template**

OLD (line 91, inside the charter code block):

```
| Expert | Bounded context | Owns | Explicitly not | Deliverables |
```

NEW:

```
| Expert | Bounded context | Owns | Explicitly not | Deliverables | Model |
```

This string must match exactly once in `skills/mtoe/SKILL.md`. (The parent design doc has the same line but that is a different file, edited in Task 7.)

- [ ] **Step 2: Extend the hard-gate paragraph**

OLD (lines 107–108):

```
**The user must approve the charter before any expert spawns.** Present it, wait for
approval, revise on rejection. Nothing spawns until approved.
```

NEW (append the Model-column paragraph after it):

```
**The user must approve the charter before any expert spawns.** Present it, wait for
approval, revise on rejection. Nothing spawns until approved.

The `Model` column carries each expert's resolved model: an explicit assignment, a
right-sized recommendation with one or two **available** alternatives (e.g.
`acme/4b-free` — well-specified scaffolding; alt: `acme/9b-free`), or `inherited`.
Invite model edits explicitly when presenting the charter: *approve the roster,
boundaries, and model assignments — or name changes?*
```

- [ ] **Step 3: Verify**

Run: `grep -c "Deliverables | Model |" skills/mtoe/SKILL.md`
Expected: `1`

- [ ] **Step 4: Commit**

```bash
git add skills/mtoe/SKILL.md
git commit -m "docs: charter roster gains a Model column at the approval gate"
```

---

### Task 4: SKILL.md Phase 4 + Phase 5 — spawn parameter and substitution reporting

**Files:**
- Modify: `skills/mtoe/SKILL.md` (Phase 4 — Delegate, Phase 5 — Integrate)

- [ ] **Step 1: Add the Model bullet to Delegate**

OLD (lines 114–116, the Prompt composition bullet):

```
- **Prompt composition** = the expert's archetype file (full text) + that expert's
  charter slice (task brief, owns/does-not-own, deliverable spec) + the **minimal
  shared context** the task requires. Nothing else — narrow context is the point.
```

NEW (insert the Model bullet right after it, before the **Parallelism** bullet):

```
- **Prompt composition** = the expert's archetype file (full text) + that expert's
  charter slice (task brief, owns/does-not-own, deliverable spec) + the **minimal
  shared context** the task requires. Nothing else — narrow context is the point.
- **Model**: spawn each subagent with its resolved model via the harness's native
  model parameter (in OpenCode, `model: "providerID/modelID"`). Experts resolved to
  `inherited` spawn **without** the parameter, leaving the harness default
  unchanged. The model is spawn plumbing, not prompt content — the charter slice is
  identical either way.
```

- [ ] **Step 2: Extend the Integrate report item**

OLD (lines 166–168):

```
4. Report with a **deliverable → expert mapping** and an **artifact → claim
   mapping** — what execution proof backs each deliverable — so the user can trace
   every artifact to its accountable expert and its verification.
```

NEW:

```
4. Report with a **deliverable → expert mapping** and an **artifact → claim
   mapping** — what execution proof backs each deliverable — so the user can trace
   every artifact to its accountable expert and its verification. Record the model
   each expert ran on; when a substitution occurred, show **requested → actual**
   (see Failure handling).
```

- [ ] **Step 3: Verify**

Run: `grep -n "spawn plumbing" skills/mtoe/SKILL.md`
Expected: exactly one hit inside Phase 4

- [ ] **Step 4: Commit**

```bash
git add skills/mtoe/SKILL.md
git commit -m "docs: delegate spawns with the resolved model; integrate reports substitutions"
```

---

### Task 5: SKILL.md — failure-handling rows

**Files:**
- Modify: `skills/mtoe/SKILL.md` (Failure handling table)

- [ ] **Step 1: Append two rows to the failure table**

OLD (line 201 — note this wording differs from the parent design doc's version):

```
| Companion skill not installed | The accountable expert falls back to its own bounded review and states the gap in its deliverable; note the missing companion in the report. |
```

NEW (append the two new rows after it):

```
| Companion skill not installed | The accountable expert falls back to its own bounded review and states the gap in its deliverable; note the missing companion in the report. |
| Assigned model unavailable at spawn (provider not connected, bad ID) | Spawn on the harness default model and record the substitution; never block the run. The report shows requested → actual. |
| No model-listing facility in the harness | Skip recommendations; resolve to archetype defaults / `inherited`, note the gap in the charter, and let the user name models manually at the gate. |
```

- [ ] **Step 2: Verify**

Run: `grep -c "requested → actual" skills/mtoe/SKILL.md`
Expected: `2` (failure row + Phase 5 report item)

- [ ] **Step 3: Commit**

```bash
git add skills/mtoe/SKILL.md
git commit -m "docs: failure handling for unavailable models and missing model discovery"
```

---

### Task 6: README — AutoGen source and feature mention

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add the feature paragraph in the roster section**

OLD (lines 90–91):

```
Add your own: copy `experts/_TEMPLATE.md` and fill in the six sections. Tune or
delete experts by editing the files — nothing regenerates or overwrites your edits.
```

NEW (append the feature paragraph after it):

```
Add your own: copy `experts/_TEMPLATE.md` and fill in the six sections. Tune or
delete experts by editing the files — nothing regenerates or overwrites your edits.

**Per-expert models:** an expert archetype may pin a model in its `model:`
frontmatter. When it doesn't, the orchestrator right-sizes per bounded context from
the models available in your setup — the smallest credible model, not the frontier
by default — and you confirm assignments in the charter's `Model` column
([AutoGen, arXiv:2310.01905](https://arxiv.org/pdf/2310.01905)).
```

- [ ] **Step 2: Add AutoGen as design source 4**

OLD (lines 112–114):

```
1. *Agents for DDD – Back and Forth* (Ricci, Ciortea, Burattini, Castellucci — EMAS 2024) — bounded contexts as agents; application/domain layer split.
2. *Domain-Specific AI Agents: Why Composition Beats One Big General Agent* (ospeks.com) — the four-layer agent anatomy; scope as permission boundary.
3. [mattpocock/skills](https://github.com/mattpocock/skills) — user-invoked vs model-invoked skill structure.
```

NEW:

```
1. *Agents for DDD – Back and Forth* (Ricci, Ciortea, Burattini, Castellucci — EMAS 2024) — bounded contexts as agents; application/domain layer split.
2. *Domain-Specific AI Agents: Why Composition Beats One Big General Agent* (ospeks.com) — the four-layer agent anatomy; scope as permission boundary.
3. [mattpocock/skills](https://github.com/mattpocock/skills) — user-invoked vs model-invoked skill structure.
4. *AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation* (Wu et al., [arXiv:2310.01905](https://arxiv.org/pdf/2310.01905)) — every agent carries its own `llm_config`; model choice is a per-agent property.
```

- [ ] **Step 3: Verify**

Run: `grep -c "2310.01905" README.md`
Expected: `2`

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: AutoGen design source and per-expert model feature mention"
```

---

### Task 7: Parent design doc — Revision 3 pointer and sync

**Files:**
- Modify: `docs/specs/2026-11-15-mtoe-design.md`

- [ ] **Step 1: Sync the charter roster template**

OLD (line 121, inside the Phase 3 charter code block):

```
| Expert | Bounded context | Owns | Explicitly not | Deliverables |
```

NEW:

```
| Expert | Bounded context | Owns | Explicitly not | Deliverables | Model |
```

- [ ] **Step 2: Append two failure-handling rows**

OLD (line 226 — note: different wording from SKILL.md's row):

```
| Companion skill not installed | The accountable expert falls back to its own bounded review and states the gap in its deliverable; the orchestrator notes the missing companion in the report. |
```

NEW:

```
| Companion skill not installed | The accountable expert falls back to its own bounded review and states the gap in its deliverable; the orchestrator notes the missing companion in the report. |
| Assigned model unavailable at spawn (provider not connected, bad ID) | Spawn on the harness default model and record the substitution; never block the run. The report shows requested → actual. |
| No model-listing facility in the harness | Skip recommendations; resolve to archetype defaults / `inherited`, note the gap in the charter. |
```

- [ ] **Step 3: Append the Revision 3 section at the end of the file**

OLD (lines 270–272, the file's final paragraph):

```
Underlying ordering — the **inverted trust pyramid**: deterministic code
constraints > runtime verification artifacts > prose rules, with prose as the
thinnest top layer.
```

NEW (append the section after it):

```
Underlying ordering — the **inverted trust pyramid**: deterministic code
constraints > runtime verification artifacts > prose rules, with prose as the
thinnest top layer.

## Revision 3 — per-expert model assignment (AutoGen)

Applied 2026-11-16. Full spec:
[`2026-11-16-per-expert-model-assignment-design.md`](2026-11-16-per-expert-model-assignment-design.md).

Each expert can run on a **specific model**, AutoGen-style (arXiv:2310.01905 — every
agent carries its own `llm_config`):

1. **Archetype frontmatter** gains an optional `model:` field (mechanism only — no
   baked IDs in shipped archetypes).
2. **Compose** resolves each expert's model in precedence order: user preference →
   archetype default → right-sized recommendation (smallest credible model for the
   bounded context, drawn from the models the harness actually lists) → `inherited`.
3. **Charter** gains a `Model` roster column; assignments are confirmed or edited at
   the existing hard gate.
4. **Delegate** passes the resolved model to the subagent spawn; `inherited` experts
   spawn without it.
5. **Fallback**: an unavailable model spawns on the harness default and the report
   shows requested → actual. No model-listing facility → recommendations are skipped
   and the gap is noted in the charter.
```

- [ ] **Step 4: Verify**

Run: `grep -c "Revision 3" docs/specs/2026-11-15-mtoe-design.md`
Expected: `1`

- [ ] **Step 5: Commit**

```bash
git add docs/specs/2026-11-15-mtoe-design.md
git commit -m "docs: revision 3 pointer in parent design doc, roster and failure-row sync"
```

---

### Task 8: Final consistency sweep

**Files:**
- None modified (verification only; fix and commit if a check fails)

- [ ] **Step 1: Spec-coverage greps**

Run each and check expected output:

```bash
grep -c "model:" skills/mtoe/experts/_TEMPLATE.md
```
Expected: `2`

```bash
grep -c "Deliverables | Model |" skills/mtoe/SKILL.md docs/specs/2026-11-15-mtoe-design.md
```
Expected: `1` per file

```bash
grep -n "2310.01905" README.md skills/mtoe/SKILL.md docs/specs/2026-11-15-mtoe-design.md
```
Expected: hits in all three files

```bash
grep -c "requested → actual" skills/mtoe/SKILL.md
```
Expected: `2`

- [ ] **Step 2: Confirm shipped archetypes untouched**

Run: `git status --short && git log --oneline -7`
Expected: clean working tree; the seven commits from Tasks 1–7; no changes under `skills/mtoe/experts/` other than `_TEMPLATE.md`

- [ ] **Step 3: Read-through check**

Read `skills/mtoe/SKILL.md` Phase 2 → Phase 3 boundary end to end: the new `### Model discovery and right-sizing` subsection must sit inside Phase 2 (after the numbered list, before `## Phase 3 — Charter`), and the charter roster table must render with 6 columns. Fix any misplacement, then commit the fix if one was needed:

```bash
git add skills/mtoe/SKILL.md
git commit -m "docs: fix model-assignment placement from final sweep"
```

---

## Self-review record

- **Spec coverage:** frontmatter field (Tasks 1), resolution order + discovery + right-sizing (Task 2), Model column + gate (Task 3), spawn param (Task 4), substitution reporting (Task 4), failure rows (Task 5), README AutoGen + mention (Task 6), parent-doc pointer (Task 7) — all spec sections map to a task.
- **Placeholder scan:** none; every edit step carries exact OLD/NEW text.
- **Consistency:** the two "Companion skill not installed" anchor rows intentionally differ per file (verified byte-exact); `requested → actual` count expectation (2) matches Tasks 4+5 both touching SKILL.md.
