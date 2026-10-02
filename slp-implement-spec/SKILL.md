---
name: slp-implement-spec
description: Implement a Matt Pocock specification and ticket graph using Supervisor-Lead-Peer governance, Paseo agent profiles, isolated worktrees and independent review.
disable-model-invocation: true
---

# SLP Implement Spec

Implement a completed /to-spec + /to-tickets graph using SLP
governance and Paseo orchestration.

The current agent acts as Lead.

Before beginning:

1. Load `slp-governance`.
2. Read `docs/agents/WORKSTATE.md`.
3. Read `docs/agents/issue-tracker.md`.
4. Read relevant GLOSSARY and ADR files.
5. Read the full spec.
6. Read all tickets and blocking relationships.
7. Read available Paseo Agent Profiles and their notes.

Do not begin ticket implementation before understanding the
complete dependency graph.

## Phase 1: Initialize

Determine:

- integration branch
- ticket graph
- ready frontier
- shared mutable surfaces
- human constraints
- current design decisions
- relevant system invariants

Write this state into:

`docs/agents/WORKSTATE.md`

## Phase 2: Detect Scope Conflicts

Before parallelizing tickets, identify overlapping mutable
surfaces.

Tickets which need to modify the same ownership scope must not
run concurrently unless one is explicitly read-only there.

Prefer serial execution when:

- same database migration
- same shared configuration
- same public interface
- same core abstraction
- same generated file
- same package manifest

## Phase 3: Select Worker Profile

Classify each READY ticket.

Use `peer-implement-quality` for:

- architecture-sensitive code
- domain logic
- persistence
- concurrency
- authentication / authorization
- cross-module behavior
- public contracts

Use `peer-debug` for investigation of existing broken behaviour.

Use `peer-cheap` for low-risk mechanical tasks.

Otherwise use `peer-implement-fast`.

Read profile notes before creating an agent.

Do not select a provider/model directly when a matching profile
exists.

## Phase 4: Acquire Ownership

Before launching a worker, update WORKSTATE:

- Ticket → RUNNING
- Owner → agent identity
- Scope → acquired

No two running agents may own the same mutable scope.

## Phase 5: Delegate

Create a Paseo worker in an isolated worktree.

The Peer prompt must include pointers, not copied context.

Prompt shape:

Goal:
<ticket goal>

Ticket:
<ticket pointer>

Spec:
<spec pointer>

Owned mutable scope:
<scope>

Read-only related scope:
<scope>

Human constraints:
<pointer to WORKSTATE section>

Current design decisions:
<pointer to WORKSTATE section>

Relevant:
- GLOSSARY
- ADR
- previous commits
- research

Instructions:

- Load slp-governance.
- Use the TDD skill for implementation.
- Stay inside owned mutable scope.
- Investigate independently.
- Do not workaround contradictory architecture assumptions.
- Return DONE, BLOCKED or CHANGE_REQUEST.

## Phase 6: Process Peer Result

### DONE

Verify:

- acceptance criteria
- relevant tests
- branch is current with integration tip
- scope stayed within ownership

Then merge via an integration/merger agent or controlled merge.

Update WORKSTATE:

- ticket → DONE
- release ownership
- record commit/evidence

Recompute ready frontier.

### BLOCKED

Update:

- ticket → BLOCKED
- blocker
- dependency graph if required

Release ownership if no further work can proceed.

### CHANGE_REQUEST

Lead evaluates evidence.

If change is inside the same ticket and ownership scope:

- update ticket
- record decision
- continue Peer

If change affects another Peer scope:

- coordinate with current owner
- create or update dependent ticket

If change affects architecture or system invariant:

- create Supervisor agent using `slp-supervisor`
- provide evidence and pointers
- request a decision, not implementation

If Supervisor identifies a Human Constraint change:

- stop affected work
- escalate to Human

Never silently convert a Human Constraint into a technical
decision.

## Phase 7: Continue Frontier

After every completed or changed ticket:

1. refresh dependency graph
2. compute READY tickets
3. check ownership conflicts
4. launch additional workers where safe

Prefer useful parallelism over maximum parallelism.

Do not launch multiple agents merely because capacity exists.

## Phase 8: Integration Verification

Once all implementation tickets are DONE:

Run:

- build
- typecheck
- focused tests
- full relevant test suite

Record evidence in WORKSTATE.

## Phase 9: Independent Review

Determine which model family performed most implementation.

Prefer an independent reviewer from another family.

If implementation was primarily GPT:
use `peer-review-gemini`.

If implementation was primarily Gemini:
use `peer-review-gpt`.

Run two independent review axes:

1. Standards review
2. Spec/acceptance review

Reviewers are read-only.

Aggregate findings.

## Phase 10: Fix Review Findings

Create a single implementation worker for accepted findings.

Any behavior-changing fix must follow TDD.

Re-run verification.

## Phase 11: Complete

Confirm:

- all tickets DONE
- no unresolved blockers
- no unresolved CHANGE_REQUEST
- no active ownership
- integration tests pass
- review findings resolved

Update WORKSTATE.

If the configured issue tracker uses PRs, continue with `/pr`.

Do not remove historical evidence from WORKSTATE until the
feature is merged.