# Goldfish Review Layer — MTOE Skill Design (Revision 6)

Date: 2026-11-17
Status: Implemented — adopted from Google's Elephant-Goldfish Model
(https://github.com/google/elephant-goldfish-model); design doc revision-history row 6.

## Problem

MTOE's Phase 5 integration proves deliverables **execute** (binary acceptance gate,
orchestrator re-runs) but never proves they were **reviewed with fresh eyes**:

1. **No fresh-context review.** The coherence reviewer and orchestrator evaluate
   deliverables while immersed in the run's own context — the same bias EGM's
   "Goldfish" reviewers exist to remove: a reader who drafted or witnessed the
   drafting accepts unstated assumptions more readily than a cold reader.
2. **No comprehension test.** The binary gate proves artifacts exist and re-run; it
   cannot detect a document whose terms are all defined but whose parts do not connect.
3. **No anti-sycophancy contract.** Reviewers are never instructed to stress-test by
   default; glazing and reflexive agreement pass unchallenged.
4. **No finding taxonomy or resolution protocol.** "One corrective round" carries no
   semantics for which findings must be fixed, which may be contested, and what
   evidence a contest requires.
5. **No convergence semantics.** Corrective rounds have no definition of "clean",
   "settled", or "re-opened"; a failed fix is indistinguishable from progress.
6. **No not-run accounting.** A reviewer or check that fails to execute can silently
   read as "passed with zero findings".
7. **No drift detection.** Charters and specs rot unexamined after the run ends.

## Source model (reverse-engineered)

EGM roles: the **Elephant** (persistent context, drafts and repairs), the **Goldfish**
(fresh-context, read-only reviewers — *"a review in the same context is not a Goldfish
review"*), the **user** (resolves trade-offs, approves). Key mechanisms adopted here:

- **Goldfish isolation**: reviewers spawn with zero conversation history; they see only
  the deliverable and an explicit reading list.
- **Paraphrase-first comprehension test**: the reviewer paraphrases *"What this
  proposes"* and *"How it works, per this document"* before listing findings; a wrong
  paraphrase is itself a finding (the paraphrase catches documents that are locally
  defined but do not connect — a term list does not).
- **Anti-sycophancy execution constraints**: stress-test by default; critique first;
  no glazing ("great", "brilliant"); no apologies when challenged; generate the
  strongest counter-argument; agreement must be earned.
- **Severity taxonomy**: 🔴 Blocker / 🟡 Major / 🟢 Nit, with stable finding ids
  (`<reviewer>-<round>-<n>`).
- **Resolution taxonomy**: `PROPOSE FIX` (valid — define the precise edit),
  `CONTEST` (critique inaccurate — **must cite evidence the reviewer could have
  read**: a file, symbol, or charter row; *"you wrote the document — a wish to
  contest is not evidence"*), `ESCALATE` (evidence lives outside every file the
  reviewer could reach — needs the user's knowledge), `DECLINE` (nit; record, don't fix).
- **Convergence loop**: a **settled** finding (contested or accepted risk) is never
  re-opened; a **fixed** finding that reappears at the same scope means the fix failed
  and **is re-opened**; a **clean round** has no unsettled Blocker/Major and no
  not-run reviewers; **converged = 2 consecutive clean rounds**; cap 5 rounds;
  **churn** (a new finding at the scope of an applied fix whose revert would resolve
  it) → ask the user.
- **Failed-reviewer accounting**: a reviewer that times out or returns no report is
  retried once, then recorded **not run** — never counted as zero findings; a not-run
  reviewer makes the round not clean.

## Decisions

1. **Where the layer lives**: Phase 5 — Integrate, as a **fresh-eyes review** step
   between the binary acceptance gate and the corrective round. Code deliverables
   still gate on runtime proof first (trust pyramid order is unchanged: deterministic
   constraints > verification artifacts > prose; a Goldfish review is a prose-layer
   instrument and never substitutes for execution proof).
2. **Reviewer scope**: one fresh-context reviewer subagent per team run (not per
   deliverable — spawn cost discipline). Input: the charter rows + deliverable
   artifacts on disk for every accepted deliverable, **never** the conversation
   transcript or expert reasoning. Document deliverables additionally get the
   paraphrase-first comprehension test. The reviewer is read-only and returns
   findings only.
3. **Corrective rounds**: unchanged budget for code deliverables (**one** corrective
   round — MTOE cost discipline), now driven by the taxonomy. For **document
   deliverables** (designs, docs, runbooks), the user may opt into an iterative
   fresh-eyes loop with the full convergence semantics (cap 5, converged = 2
   consecutive clean rounds).
4. **Not-run rule** joins the verification contract for every participant: a failed,
   timed-out, or missing expert/check/reviewer is recorded `not run` and blocks
   `complete` until the user accepts the recorded disclosure (e.g. degraded mode) —
   it is never a pass.
5. **Drift detection**: documented as a later extension (post-run re-check of charter
   and specs against the code), plus an optional follow-up offer in the final report.
6. **Rejected adaptations** (over-engineering for a single-skill repo): per-reviewer
   swappable standard files (our archetypes embed their standards), the five-stage
   doc-review pipeline, per-section drafting gates, and multi-harness plugin manifests.

## Changes

- `skills/mtoe/SKILL.md`: fresh-eyes review step + finding/resolution taxonomy +
  convergence semantics + not-run rule in Phase 5; anti-sycophancy block in the
  subagent envelope §5; charter Integration & acceptance row; failure-handling rows;
  degraded-mode not-run disclosure.
- `docs/specs/2026-11-15-mtoe-design.md`: revision-history row 6; Integration &
  verification section sync; failure rows sync; Later extensions (drift report,
  multi-harness distribution).
- `README.md`: EGM as design source #6; fresh-eyes review mentioned in the
  verification bullet.

## Verification

- Grep gates: `fresh-eyes` present in SKILL.md Phase 5; `CONTEST`/`ESCALATE`/
  `PROPOSE FIX`/`DECLINE` present; `not run` rule present; paraphrase headings
  present; convergence (2 clean rounds / cap 5) present.
- No existing gate weakened: binary acceptance gate conditions unchanged; wave-0,
  write-ownership, and containment rules untouched.
- Reviewer subagent passes: nothing from rev 3–5 lost; EGM vocabulary consistent.
