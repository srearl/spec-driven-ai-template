# Spec-Driven, Well-Architected Workflow for R/Python + EML/EDI

A reusable VS Code + GitHub Copilot setup that turns ad-hoc "vibe" prompting into
an intentional engineering process. Specifications are version-controlled and
reviewed **before** code — "specs as Infrastructure-as-Code." Every non-trivial
decision is scored against Well-Architected pillars and grounded in authoritative
docs, not model memory.

## Why this exists

Three habits make AI-assisted work reliable: (1) ground answers in authoritative
docs, (2) force a design step that weighs options, (3) write an action plan
before implementing. This repo wires each habit to a native VS Code primitive so
the flow is repeatable instead of fiddly.

## The Loop

`Frame → Architect → Specify → Plan → Implement → Verify`

1. **Frame** — restate the request as goals, non-goals, constraints, success criteria.
2. **Architect** — weigh options against the pillars using real docs (`Architect` agent).
3. **Specify** — `/spec` writes `specs/<slug>/spec.md`. Review & commit.
4. **Plan** — `/plan` writes `specs/<slug>/plan.md` (`Planner` agent). Review & commit.
5. **Implement** — execute the plan; if reality diverges, update the spec first.
6. **Verify** — tests/linters; confirm the spec's success criteria.

## How to use it

The whole flow happens in the VS Code Chat view. Slash commands (`/spec`,
`/plan`) come from `.github/prompts/`; agents are chosen in the chat agent
picker. Each step ends with a **review + commit** so the spec/plan stays the
source of truth.

### 1. Architect (optional, for non-trivial work)
Pick the **Architect** agent in the chat picker and describe the problem. It is
read-only — it researches and recommends, never edits.

> Architect: We need to publish a multi-year stream-chemistry CSV to EDI.
> Compare options for generating the EML and recommend one.

It returns options scored against `docs/waf/pillars/`, a recommendation, and
sources. Skip this for small, obvious tasks and go straight to `/spec`.

### 2. Specify
Run the prompt; it writes `specs/<slug>/spec.md`, then stop to review.

```text
/spec publish the stream-chemistry CSV to EDI
```

```bash
git add specs/eml-stream-chemistry/spec.md
git commit -m "spec: stream-chemistry EDI package"
```

### 3. Plan
Hand the approved spec to `/plan` (uses the **Planner** agent); review the steps.

```text
/plan specs/eml-stream-chemistry/spec.md
```

```bash
git add specs/eml-stream-chemistry/plan.md
git commit -m "plan: stream-chemistry EDI package"
```

### 4. Implement
Switch to the default agent and work the plan step by step. Language and domain
instructions auto-apply (e.g. editing `make.R` loads `r-style`; metadata work
loads the `eml-metadata` skill). If reality diverges, update the spec first.

```text
Implement step 5 of specs/eml-stream-chemistry/plan.md
```

### 5. Verify & log
Confirm the spec's success criteria, then record any operating-model change.

```bash
# R example: rebuild + validate from a clean state
Rscript make.R && Rscript -e 'EML::eml_validate("eml/out.xml")'
```

```text
Add a CHANGELOG.md entry: added EDIutils publish step; why = staging review.
```

## When to use: build software vs. analyze data

Same base, two modes — pick by **deliverable**, not by repo:

| | Build software | Analyze data |
|---|---|---|
| Deliverable | Code + tests | Validated result / dataset / report |
| Spec problem | Feature/fix | Research question |
| Success criteria | Tests pass | Metric/threshold met, assumptions checked |
| Reliability means | Uptime, recovery | **Reproducibility** (seed, lockfile, clean rebuild) |
| Extra layer | language style | `analysis` instruction + `targets`/Snakemake; `eml-metadata` to package |

You don't need a separate template — the analysis path just loads
`analysis.instructions.md` and, for deposit, the `eml-metadata` skill.

## Resources: what each does, when it loads, when to tweak

| Resource | Used by AI when… | Tweak when… |
|---|---|---|
| `.github/copilot-instructions.md` | Always — base loop + pillars | Loop or pillars change |
| `instructions/r-style`, `python-style` | Editing matching files (`applyTo`) | Tooling/linters change |
| `instructions/eml-metadata` | EML/EDI task detected | EDI workflow changes |
| `instructions/analysis` | Analysis/modeling task | Pipeline conventions change |
| `instructions/well-architected` | Design decision; points to `docs/waf/` | New pillar guidance |
| `prompts/spec.md` (`/spec`), `plan.md` (`/plan`) | You invoke them | Spec/plan structure changes |
| `agents/Architect` (read-only) | Up-front design/research | Model or tools change |
| `agents/Planner` (read-only) | Spec → plan | Plan format changes |
| `skills/eml-metadata/` | Building/validating EML | Schema/EDI/ROpenSci updates |
| `docs/waf/pillars/` | Cited during Architect/spec | Refine checklists |
| `specs/` | Source of truth read at implement | Per task (its content) |
| `.vscode/mcp.json` | Live external docs (context7, MS Learn) | Add doc sources |
| `.github/CHANGELOG.md` | Reference for prior decisions | Every model change |

**Don't** wrap in-workspace folders (`docs/waf`, `specs`) in filesystem MCP
servers — the agent reads them natively; read-only is enforced via agent `tools:`.
Use MCP only for live/external docs.

## PostgreSQL workflow (recommended)

For database-heavy R/Python projects, use the PostgreSQL instruction set in
`.github/instructions/postgresql.instructions.md` and prefer MCP-backed DB
operations in VS Code.

- Why MCP-first: safer read-first workflow, schema/context inspection before
	changes, easier query-plan and performance diagnostics.
- Suggested sequence: inspect context -> run read-only queries -> apply reviewed
	modifications.
- Keep SQL in version control (`*.sql`) so PostgreSQL instructions auto-apply.

### PostgreSQL MCP quick start

1. Connect with your saved profile and select the target database.
2. Fetch schema context before changes (tables, indexes, functions).
3. Run a read-only query first to validate assumptions.
4. For slow queries, capture query plan/metrics before optimizing.
5. Apply reviewed modifications and re-check context/row counts.

## Reuse across projects

Lift the language-agnostic base (instructions, agents, prompts, `docs/waf`) into
your user profile so it roams; keep domain skills (`eml-metadata`) and `specs/`
in-repo. See `specs/README.md` and `specs/eml-stream-chemistry/` for a worked example.
