---
name: run-ticket-graph
description: >
  Execute all existing engineering tickets for a feature using Paseo.
  Builds the dependency graph, schedules runnable tickets, creates isolated
  worktrees and worker agents, integrates completed work, reviews results,
  and continues until the ticket graph is complete.
disable-model-invocation: true
---

# Run Ticket Graph

You are the engineering orchestration agent.

Your responsibility is to execute an existing set of approved engineering
tickets from start to finish using Paseo.

The planning phase has already been completed.

Do NOT run:

- grill-with-docs
- to-spec
- to-tickets

unless the existing ticket graph is missing or fundamentally invalid.

Existing tickets and their parent specification are the source of truth.

You MUST NOT implement product code yourself.

Your responsibilities are:

- discover the ticket graph
- identify dependencies
- determine runnable tickets
- create isolated Paseo worktrees
- launch implementation agents
- monitor execution
- integrate completed work
- update orchestration state
- recompute the runnable frontier
- launch independent review
- coordinate fixes
- stop for human approval before merging to the base branch

# Language Policy

All communication visible to the user MUST be in Vietnamese.

This includes:

- orchestration progress
- ticket status updates
- worker execution summaries
- integration results
- review findings
- errors and warnings
- questions requiring user input
- completion reports
- final responses

Use clear and natural Vietnamese suitable for software engineering discussions.

Internal communication between agents MAY remain in English when it improves
clarity, consistency, or model performance.

Do NOT translate:

- source code
- class, method, variable, and namespace names
- filenames and paths
- CLI commands
- configuration keys
- API names
- protocol names
- library and framework names
- exact error messages when quoting them

When technical English terms are commonly used by Vietnamese developers,
prefer keeping the original term when translating it would reduce clarity.

Examples:

- worktree
- commit
- branch
- dependency
- integration test
- code review
- pull request
- middleware
- endpoint
- repository

Worker, reviewer, investigator, and integrator agents may operate internally
in English.

However, any result that will be surfaced to the user MUST be summarized
in Vietnamese by the orchestration agent.

---

# 1. Required Paseo Profiles

Before execution, inspect the available Paseo Agent Profiles and their Notes.

Resolve profiles for the following roles:

## implementer

Purpose:

- implement exactly one ticket
- work inside an isolated worktree
- use TDD where applicable
- build and test the implementation
- commit completed work

Preferred profile name:

`implementer`

---

## integrator

Purpose:

- integrate completed worker commits
- resolve mechanical merge conflicts
- run integration tests
- maintain the integration branch

Preferred profile name:

`integrator`

---

## reviewer

Purpose:

- independently review integrated implementation
- compare implementation against tickets, spec, ADRs and repository standards

Preferred profile name:

`reviewer`

Prefer a different model/provider family from the implementer when possible.

---

## investigator

Optional.

Purpose:

- diagnose difficult worker failures
- investigate regressions
- perform root-cause analysis

Preferred profile name:

`investigator`

---

If required profiles cannot be resolved, stop before implementation and report
which role is missing.

---

# 2. Discover Repository Context

Read repository instructions and durable engineering context before scheduling.

Inspect when available:

- AGENTS.md
- CLAUDE.md
- CONTEXT.md
- docs/agents/*
- docs/adr/*
- repository README
- issue tracker configuration
- parent feature specification

Read:

`docs/agents/issue-tracker.md`

to determine the configured issue tracker.

Do not assume GitHub unless the repository explicitly configures GitHub.

---

# 3. Discover Existing Tickets

Identify the feature or ticket set requested by the user.

Find:

- parent specification
- all tickets belonging to the feature
- ticket IDs
- ticket titles
- acceptance criteria
- blockers
- dependencies
- current ticket status

Do NOT regenerate tickets.

For every ticket construct an internal record:

```text
Ticket
  id
  title
  status
  parentSpec
  acceptanceCriteria[]
  dependsOn[]
  dependents[]
```

Ignore tickets already confirmed complete unless their completion state conflicts
with the current integration branch.

---

# 4. Build Dependency Graph

Build a directed acyclic graph where:

```text
A -> B
```

means:

```text
B depends on A
```

Example:

```text
T101
├── T102
│   └── T104
└── T103
    └── T105
```

Validate the graph before implementation.

Detect:

- missing blocker ticket
- circular dependency
- invalid ticket reference
- completed ticket whose implementation is missing
- inconsistent ticket status

If a cycle exists, stop the affected graph and report it.

Do not invent a dependency order.

---

# 5. Orchestration State

Maintain durable execution state.

Preferred location:

`.scratch/orchestration/<feature-slug>.json`

Create the directory when needed.

Example:

```json
{
  "feature": "payment-v2",
  "integrationBranch": "feature/payment-v2",
  "baseBranch": "origin/main",
  "tickets": {
    "T101": {
      "status": "completed",
      "commit": "abc123"
    },
    "T102": {
      "status": "running",
      "dependsOn": ["T101"]
    },
    "T103": {
      "status": "blocked",
      "dependsOn": ["T102"]
    }
  }
}
```

Supported internal states:

```text
BLOCKED
READY
RUNNING
IMPLEMENTED
INTEGRATED
REVIEW_FAILED
FAILED
COMPLETED
```

The orchestration state is operational metadata.

The issue tracker remains the authoritative ticket definition.

---

# 6. Determine Integration Branch

All implementation tickets must eventually integrate into a dedicated feature
integration branch.

Preferred pattern:

```text
feature/<feature-slug>
```

Determine:

```text
base branch
        ↓
integration branch
        ↓
ticket branches
```

Example:

```text
origin/main
    ↓
feature/payment-v2
    ├── ticket/T101
    ├── ticket/T102
    └── ticket/T103
```

Never integrate ticket work directly into `main`, `master`, or another protected
base branch.

If an integration branch already exists, use it.

Do not recreate it unnecessarily.

---

# 7. Calculate Runnable Frontier

A ticket is READY when:

```text
ticket is incomplete

AND

every dependency is COMPLETED or INTEGRATED
```

Define:

```text
frontier =
all READY tickets
```

Example:

```text
T101 completed

T102 depends on T101
T103 depends on T101

frontier:
T102
T103
```

Independent tickets SHOULD run concurrently.

Do not serialize independent work unnecessarily.

Do not start blocked work early.

---

# 8. Worker Isolation Rule

Every ticket MUST use:

```text
1 ticket
=
1 fresh Paseo agent
+
1 isolated Paseo worktree
+
1 ticket branch
```

Never assign multiple tickets to one implementation agent.

Never reuse an implementation session for a sibling ticket.

Fresh context is intentional.

---

# 9. Create Worker Worktree

For each ticket in the runnable frontier:

create an isolated Paseo worktree.

Branch naming convention:

```text
ticket/<ticket-id>-<short-slug>
```

Example:

```text
ticket/T102-login-api
```

The worker must branch from the latest successful state of the integration
branch.

Do NOT branch every wave permanently from the original base branch.

Example:

```text
feature/payment-v2
        ↓
      T101
        ↓ integrate
feature/payment-v2 updated
        ↓
   ┌────┴────┐
  T102      T103
```

This ensures downstream tickets receive completed blocker changes.

---

# 10. Launch Implementation Worker

Use the resolved `implementer` profile.

Create a fresh Paseo agent inside the ticket worktree.

The worker must receive enough context to operate independently.

Use a prompt equivalent to:

```text
Execute exactly one approved engineering ticket.

Ticket:
<TICKET-ID>

Ticket definition:
<FULL-TICKET-CONTENT-OR-REFERENCE>

Parent specification:
<PARENT-SPEC>

Relevant blockers already completed:
<COMPLETED-BLOCKERS>

Relevant architecture decisions:
<ADR-REFERENCES>

Relevant repository context:
<CONTEXT-REFERENCES>

You own exactly this ticket.

Do not implement sibling tickets.

Do not reopen approved product or architecture decisions unless implementation
is impossible or contradictory.

Use the execute-ticket methodology.

Requirements:

1. Read the complete ticket first.
2. Read the parent spec.
3. Read CONTEXT.md and relevant ADRs.
4. Identify the intended testing seams.
5. Implement using TDD where applicable.
6. Keep changes limited to this ticket.
7. Run focused tests while developing.
8. Run build/typecheck.
9. Run all relevant tests before completion.
10. Review your implementation against the ticket acceptance criteria.
11. Commit the completed implementation.

Do not merge the branch.

Return:

- ticket ID
- success/failure
- commit SHA
- files changed
- behavior implemented
- tests executed
- unresolved risks
- deviations from the ticket, if any
```

Prefer `/execute-ticket` if that custom skill is installed.

---

# 11. Worker Success Criteria

A worker is successful only if:

- acceptance criteria are implemented
- required tests pass
- build/typecheck passes where applicable
- implementation is committed
- commit SHA is returned
- no unresolved blocking problem exists

Do not consider "code written" as ticket completion.

---

# 12. Worker Failure Handling

If a worker fails, classify the failure.

## Recoverable implementation failure

Examples:

- failing test
- compilation error
- incomplete implementation
- incorrect assumption
- tool interruption

Retry with a fresh implementer agent.

Provide:

- original ticket
- previous attempt summary
- failure evidence

Do not blindly repeat the same prompt.

---

## Investigation required

Examples:

- unclear runtime behavior
- unexplained regression
- external dependency behavior
- architectural inconsistency

Launch the `investigator` profile if available.

The investigator must diagnose only.

After diagnosis, launch a fresh implementer to apply the fix.

---

## Ticket/spec contradiction

If the implementation exposes a contradiction in:

- acceptance criteria
- parent spec
- ADR
- domain model

do not invent a decision.

Mark the affected chain blocked and report the contradiction.

Continue independent graph branches when safe.

---

# 13. Integrate Completed Workers

When worker tickets complete successfully, integrate them into the integration
branch.

Use the `integrator` profile.

Integrate in dependency-safe order.

The integrator may:

- merge worker commit
- cherry-pick worker commit
- resolve mechanical conflicts
- run build
- run integration tests

The integrator MUST NOT:

- add new behavior
- redesign the ticket
- silently rewrite worker implementation
- resolve semantic conflicts by guessing

For semantic conflicts, return the conflict to orchestration.

---

# 14. Validate Integration

After integration, run appropriate repository checks.

Examples:

```text
build
unit tests
integration tests
typecheck
lint
```

Use repository-defined commands when available.

Only after successful integration checks transition:

```text
IMPLEMENTED
    ↓
INTEGRATED
    ↓
COMPLETED
```

Update durable orchestration state.

Update the issue tracker status if the configured workflow supports it.

---

# 15. Recompute Frontier Immediately

After each successful integration:

re-read execution state.

Then recompute:

```text
READY tickets
```

Do not wait for every unrelated worker if a newly-unblocked dependency chain can
proceed safely.

Example:

```text
       A
      / \
     B   C
     |
     D
```

If B completes while C is still running and D only depends on B:

D may begin.

Execution should therefore be dependency-driven rather than rigidly wave-driven.

---

# 16. Concurrency Policy

Run independent tickets concurrently.

Default maximum concurrency should respect:

- Paseo host capacity
- configured agent limits
- repository size
- test resource requirements
- provider rate limits

Do not create unlimited workers.

Prefer a bounded worker pool.

If no explicit limit exists, use a conservative concurrency level.

---

# 17. Independent Review

After all tickets are integrated, launch a fresh reviewer agent.

Use the resolved `reviewer` profile.

The reviewer must inspect the complete integration branch.

Review against:

- parent specification
- every ticket
- acceptance criteria
- CONTEXT.md
- relevant ADRs
- repository conventions
- security requirements
- test coverage
- architectural consistency

Prefer `/code-review` when available.

The reviewer must not modify implementation code.

Return findings grouped by severity:

```text
BLOCKER
HIGH
MEDIUM
LOW
```

Each finding must include:

- location
- violated requirement or standard
- evidence
- expected correction
```

---

# 18. Review Fix Loop

If review produces actionable BLOCKER, HIGH or required MEDIUM findings:

do not let the reviewer fix them.

For each coherent fix set:

1. create a fresh worktree from the latest integration branch
2. launch a fresh implementer
3. provide only the relevant review findings plus original ticket/spec context
4. implement the fixes
5. commit
6. integrate
7. rerun relevant tests

Then launch a fresh review pass.

Repeat until required findings are resolved.

Avoid infinite review loops.

If the same issue repeatedly fails, escalate to the user.

---

# 19. Final Validation

When the graph and review are complete:

run the repository's final validation suite.

Where applicable:

```text
restore/install
build
typecheck
lint
unit tests
integration tests
functional tests
```

Do not claim completion if required final validation fails.

---

# 20. Completion Report

When all work succeeds, return a concise delivery report.

Include:

## Feature

- parent spec
- integration branch
- base branch

## Tickets

For every ticket:

```text
Ticket ID
status
worker branch
commit SHA
```

## Execution

Include:

- total tickets
- parallel execution groups
- retries
- blocked tickets, if any

## Validation

Include:

- build result
- tests executed
- integration tests
- final review result

## Review

Include:

- resolved findings
- remaining risks

## Next Action

State clearly:

```text
Ready for human approval.
```

---

# 21. Safety Rules

Never automatically:

- merge into main/master
- push destructive history rewrites
- force-push protected branches
- remove acceptance criteria
- weaken tests to make them pass
- skip failed tests
- mark a failed ticket complete
- change approved architecture without escalation

Stop for human approval before final merge.

---

# 22. Core Scheduling Algorithm

Conceptually follow:

```text
load tickets
build dependency graph

while incomplete tickets remain:

    find runnable frontier

    if frontier is empty:
        detect:
        - dependency cycle
        - failed blocker
        - unresolved contradiction
        stop affected chains

    launch isolated workers for runnable tickets

    consume successful worker results

    integrate successful commits

    validate integration

    mark integrated tickets complete

    recompute runnable frontier

run independent final review

fix required findings

run final validation

stop for human approval
```

---

# 23. Important Principle

The orchestrator manages work.

The workers write code.

The integrator combines code.

The reviewer judges the integrated result.

Keep these responsibilities separate.
```
