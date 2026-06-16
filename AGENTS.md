# AGENTS.md

Cross-cutting rules read by every agent on every turn (declared in `opencode.jsonc` as `instructions`).

## Skill Routing Matrix

Mrbrain owns the orchestration of skill loading. Each delegated agent loads the skills named in its directive. The matrix below says **which agent loads which skill for which task pattern**. The agent that loads a skill applies it to its own work — skills are not transitive.

### Architecture & decisions

| Task pattern | Skill to load | Agent that loads it |
| --- | --- | --- |
| Recording or referencing an ADR | `architecture-decision-records` | Mrbrain (Observe/Think/Plan) |
| Choosing layer boundaries, Clean/Hexagonal/Onion, DDD bounded contexts | `architecture-patterns` | Mrbrain (Observe/Think/Plan) |
| Mapping existing patterns in a codebase before implementation | `architecture-patterns` | Mrscout (Build — Scout phase) |
| Reviewing code for layer-boundary violations | `architecture-patterns` | Mrsreview (Verify) |

### Backend & data

| Task pattern | Skill to load | Agent that loads it |
| --- | --- | --- |
| Implementing backend code (handler/service/repo, error handling, idempotency, caching) | `backend-patterns` | codyy (Execute) |
| Reviewing backend code for pattern violations | `backend-patterns` | Mrsreview (Verify) |
| Mapping existing backend patterns in a codebase | `backend-patterns` | Mrscout (Build) |
| Designing or changing PostgreSQL schema, indexes, constraints, query shape | `postgresql-table-design` | codyy (Execute) |
| Schema-change decision rationale (index choice, type choice) | `postgresql-table-design` | Mrbrain (Plan) |
| Hard concurrency / write-path safety analysis | `postgresql-table-design` | spock (specialist) |

### APIs

| Task pattern | Skill to load | Agent that loads it |
| --- | --- | --- |
| Designing a new REST/GraphQL API contract | `api-design-principles` | codyy (Execute) |
| Reviewing an API surface for design issues | `api-design-principles` | Mrsreview (Verify) |
| Mapping existing API conventions in a codebase | `api-design-principles` | Mrscout (Build) |

### Distributed systems

| Task pattern | Skill to load | Agent that loads it |
| --- | --- | --- |
| Microservice decomposition, service boundaries, inter-service comms | `microservices-patterns` | Mrbrain (Plan) or spock (specialist) |
| Deciding to split read/write models (CQRS) | `cqrs-implementation` | Mrbrain (Plan) or spock (specialist) |
| Designing read models / projections from event streams | `projection-patterns` | codyy (Execute) |
| Designing an event store / event-sourced aggregate | `event-store-design` | codyy (Execute) |
| Saga / distributed transaction / compensating actions | `saga-orchestration` | Mrbrain (Plan) or spock (specialist) |
| Durable workflow (Temporal, activity vs workflow) | `workflow-orchestration-patterns` | Mrbrain (Plan) or spock (specialist) |
| Documenting a distributed system as a component diagram | `microservices-patterns` | data (Execute, DIAGRAM DIRECTIVE) |

### Frontend (web)

| Task pattern | Skill to load | Agent that loads it |
| --- | --- | --- |
| Building a page, landing, dashboard, or full UI | `frontend-design` | codyy (Execute) |
| Building a reusable component (React/Vue/Svelte) | `web-component-design` | codyy (Execute) |
| Working with MUI v7 components, `sx` prop, theme | `mui` | codyy (Execute) |
| Establishing design tokens, theming, or component-library foundation | `design-system-patterns` | codyy (Execute) |
| Typography, color, spacing, iconography decisions | `visual-design-foundations` | codyy (Execute) |
| Responsive layout (container queries, fluid type, breakpoints) | `responsive-design` | codyy (Execute) |
| Microinteractions, motion, loading states, transitions | `interaction-design` | codyy (Execute) |

### Diagrams

| Task pattern | Skill to load | Agent that loads it |
| --- | --- | --- |
| Producing a PlantUML component/sequence diagram from a Scout Report | (none — diagram format rules live in `codyy.md` and `data.md`) | data |
| Mapping shared packages, dependencies, layer structure for a diagram | `architecture-patterns` | Mrscout (Build) — feeds data |

## Skill loading discipline

1. **Load only what the current phase needs.** A skill loaded out of phase burns context. Mrbrain loads decision-time skills during Observe/Think/Plan; codyy loads implementation skills during Execute; Mrsreview loads review-time skills during Verify; Mrscout loads discovery skills during Build.
2. **Skills are not transitive.** If Mrbrain loads `architecture-patterns` for an Observe decision, codyy does not automatically get that context. Mrbrain must name the skill in the BUILDER DIRECTIVE if codyy should also load it.
3. **One skill at a time when the topic is narrow; multiple when the task spans domains.** A pure-PostgreSQL schema change is `postgresql-table-design` only. A new API endpoint that touches the schema is `api-design-principles` + `postgresql-table-design` + `backend-patterns` — name all three in the directive.
4. **Trust the skill's own scope.** Each skill's `description` field tells you when to load it. If the description does not match the current task, do not load it.
5. **Skill failures are non-blocking.** If a skill cannot be loaded (file missing, path issue), proceed with general practice and flag it in the BUILDER/REVIEWER REPORT under "Issues encountered."

## Cross-cutting orchestration rules

- **No agent self-executes.** Mrbrain does not write source files or session artifacts. Use codyy for multi-file source, fourfi for trivial mechanical changes, artifacty for session artifacts.
- **Mrscout always before codyy.** Never send codyy into a codebase without Mrscout findings — even for "mechanical" tasks.
- **Mrsreview always before any PR.** No exemption for trivial tasks.
- **Output format sections are non-negotiable structure.** Each agent's REPORT format (SCOUT REPORT, BUILDER REPORT, REVIEW REPORT, SPECIALIST REPORT, DIAGRAM REPORT, SCRIBE CONFIRMATION, UHURA REPORT) must be followed verbatim. Do not summarize, do not reformat, do not omit required fields.

## Branch isolation

Every session operates on a dedicated branch. Confirm the branch exists before delegating. If it does not exist: `git switch -c {branch-name}`. Never write to `main`. Never instruct any agent to commit to `main`.

## Session close hard gates

Session close is BLOCKED until all of the following are confirmed with evidence:

- [ ] `execute.md` documents each ISC criterion as satisfied with specific evidence - not placeholder text
- [ ] `verify.md` records Mrsreview verdict (PASS or PASS_WITH_NOTES) with full ISC verification table - not placeholder
- [ ] Agent Dispatch Log in `session.md` is complete - every row updated to `invoked` or `skipped` with rationale
- [ ] Phase Log in `session.md` is complete - all 7 phases stamped
- [ ] `calibration-notes.md` entry written using feedback schema
- [ ] No open ESCALATED items
- [ ] `handoff.md` written by artifacty
