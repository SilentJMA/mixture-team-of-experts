# Mixture Team of Experts (MTOE)

A coding-agent skill that turns a request into a **team of domain-specific experts**
and orchestrates them — instead of handing everything to one general-purpose agent.

```
        your request
             │
        ┌────▼─────┐
        │ orchestrator │   application layer: triage, compose,
        └────┬─────┘   charter, delegate, integrate
   ┌─────────┼──────────┬─────────────┐
   ▼         ▼          ▼             ▼
 backend   frontend   ai/llm     security/rea/…
 expert    expert     expert        experts
   └─ bounded contexts: narrow context, explicit boundaries,
      deliverables handed back as artifacts
```

- **Experts are bounded contexts.** Each one owns a domain, speaks its own language,
  and refuses what is outside it (DDD × agent-oriented software engineering —
  [*Agents for DDD – Back and Forth*, EMAS 2024](https://emas.in.tu-clausthal.de/2024/assets/papers/EMAS_2024_paper_7.pdf)).
- **Composition beats one big general agent.** Narrow context is the permission
  boundary; specialists fail in one place, parallelize cleanly, and cost fewer
  tokens ([*Domain-Specific AI Agents: Why Composition Beats One Big General Agent*](https://ospeks.com/domain-specific-ai-agents-composition-vs-general-agent/)).
- **Structure follows [mattpocock/skills](https://github.com/mattpocock/skills)**:
  one user-invoked orchestrator, a library of model-invoked expert archetypes,
  plain markdown you can hack on.

## Install

```bash
# project-level (editable files copied into your project)
npx skills add https://github.com/SilentJMA/mixture-team-of-experts --skill mtoe

# or user-level, available in every project
npx skills add https://github.com/SilentJMA/mixture-team-of-experts --skill mtoe --global
```

**Manual install:** copy `skills/mtoe/` into your agent's skill directory — e.g.
`~/.agents/skills/mtoe` (global) or `<project>/.claude/skills/mtoe` (per-project).
No dependencies beyond the markdown files.

## Use

Invoke `/mtoe` (or just say *"put a team of experts on this"*) with your request:

```
/mtoe add OAuth login to our app — Google + GitHub, sessions, and an admin audit trail
```

The orchestrator then runs five phases:

1. **Triage** — goal, deliverable type, risk domains, constraints (asks up to 3 questions if vague).
2. **Compose** — the smallest team (2–5 experts) that covers the risk domains; each expert gets a bounded context.
3. **Charter** — a `team-charter.md` (roster, interfaces, execution waves, verification). **You approve it before anything spawns.**
4. **Delegate** — one subagent per expert, each seeing only its archetype + charter slice; independent experts run in parallel.
5. **Integrate** — deliverables checked against the charter, conflicts resolved by the orchestrator, every artifact traced to its accountable expert.

## The roster

| Archetype | Bounded context |
|---|---|
| `product-strategist` | Scope, priorities, acceptance criteria (the integrating lead) |
| `backend-architect` | API contracts, data model, invariants, concurrency |
| `frontend-engineer` | UI, state, accessibility, i18n/RTL, responsiveness |
| `ai-llm-engineer` | Retrieval, embeddings, rerank, prompts, evaluation |
| `devops-sre` | Deployment, CI/CD, observability, runbooks |
| `security-compliance` | Threat models, findings, data-protection requirements |
| `reverse-engineer` | Feature archaeology of a target app, without source |
| `data-engineer` | Ingestion, sync/watermarks, dedup, migrations |
| `qa-test-engineer` | Test strategy, seams, fixtures, regression nets |

Add your own: copy `experts/_TEMPLATE.md` and fill in the six sections. Tune or
delete experts by editing the files — nothing regenerates or overwrites your edits.

## Companion skills

When installed, MTOE routes specialized work through them — only via the
accountable expert:

- **[cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill)** —
  the `security-compliance` expert runs explicit audit requests through its
  six-phase, adversarially-validated workflow (`confirmed` / `needs_validation` /
  `rejected` findings). Security *questions* stay in guidance mode.
- **[morluto/rea](https://github.com/morluto/rea)** — when you ask to *analyze a
  similar app to implement a similar feature*, the `reverse-engineer` expert uses
  REA's MCP tools to inspect the target and returns evidence-backed findings as a
  handoff artifact for the implementing experts.

Missing companions degrade gracefully: the expert does a bounded review itself and
flags the gap.

## Design sources

1. *Agents for DDD – Back and Forth* (Ricci, Ciortea, Burattini, Castellucci — EMAS 2024) — bounded contexts as agents; application/domain layer split.
2. *Domain-Specific AI Agents: Why Composition Beats One Big General Agent* (ospeks.com) — the four-layer agent anatomy; scope as permission boundary.
3. [mattpocock/skills](https://github.com/mattpocock/skills) — user-invoked vs model-invoked skill structure.

Full design: [`docs/specs/2026-11-15-mtoe-design.md`](docs/specs/2026-11-15-mtoe-design.md).

## License

MIT.
