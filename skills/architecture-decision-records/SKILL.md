---
status: approved
name: architecture-decision-records
source: everything-claude-code
domain: process
description: >
  ADR workflow and conventions for tracking architectural decisions.
  Adapted from everything-claude-code for neobank context.
---

# Architecture Decision Records (ADRs)

## ADR Lifecycle

### Statuses
- **Draft**: Proposed, under discussion. Strong guidance but not enforceable.
- **Accepted**: Approved and enforceable. All code must comply.
- **Superseded**: Replaced by a newer ADR. Reference the replacement.
- **Deprecated**: No longer relevant. Keep for historical context.

### Creation Process
1. Identify a recurring decision point or architectural constraint
2. Draft ADR with context, decision, and consequences
3. Review with team - human sign-off required
4. Accept and enforce in Reviewer checklist

## ADR Format

```markdown
# ADR-{NNN}: {Title}

## Status
{draft | accepted | superseded | deprecated}

## Context
What is the issue that we're seeing that is motivating this decision?

## Decision
What is the change that we're proposing and/or doing?

## Consequences
What becomes easier or harder because of this change?

## Compliance
How is this enforced? (Reviewer checklist, CI, manual review)
```

## Current Active ADRs (pablo-core)

| ID | Title | Status | Enforcement |
|----|-------|--------|-------------|
| ADR-002 | Permission-based authorization | Accepted | Reviewer checklist item |
| ADR-006 | Supply chain - no new deps without review | Accepted | Reviewer checklist + human gate |
| ADR-007 | Structured JSON logging | Draft | Reviewer guidance (not hard rule) |

## Agent Integration
- Lead references ADRs when decomposing tasks
- Builder receives relevant ADR constraints in directives
- Reviewer checks compliance against active ADR checklist
- New ADRs require human creation - agents cannot create ADRs
