---
name: slp-governance
description: Governance rules for Supervisor-Lead-Peer multi-agent engineering. Use whenever coordinating delegated engineering work, assigning ownership, escalating design changes, or reporting ticket outcomes.
---

# SLP Governance

This skill defines the governance rules for multi-agent engineering.

It does not replace engineering skills such as TDD, debugging,
research or code review.

Those skills define HOW work is done.

This skill defines WHO owns work, WHO may change scope, and HOW
decisions are escalated.

## Roles

### Human

Owns:

- product behaviour
- business requirements
- budget
- delivery priorities
- security policy
- externally committed contracts
- irreversible trade-offs

Only the Human may change a Human Constraint.

### Supervisor

Owns cross-scope technical governance.

Use Supervisor for:

- architecture changes
- system invariant changes
- cross-module conflicts
- persistence architecture
- security boundaries
- distributed consistency
- public API redesign
- irreversible technical decisions

Supervisor does not implement ordinary tickets.

### Lead

Lead is the persistent owner of the implementation effort.

Lead owns:

- WORKSTATE
- ticket graph
- ready frontier
- ownership allocation
- dependency coordination
- integration
- local scope conflict resolution
- escalation

Lead should delegate implementation rather than perform
Peer-owned work.

### Peer

Peers own bounded technical scopes.

Peers may independently investigate and make implementation
decisions inside their scope.

Peers must not silently modify another Peer's owned scope.

## Ownership Invariant

At any moment each mutable scope has at most one owner.

Reading outside owned scope is allowed.

Writing outside owned scope requires one of:

1. ownership transfer by Lead
2. explicit temporary authorization by Lead
3. a new ticket owned by the requesting Peer

## Constraints vs Design Decisions

Human Constraints are immutable to agents.

Examples:

- maintain API v1 compatibility
- no data loss
- must support PostgreSQL

Current Design Decisions are reopenable.

Examples:

- use Redis for cache
- use REST between two internal modules
- store the value in table X

If codebase evidence contradicts a Current Design Decision,
raise CHANGE_REQUEST.

Do not silently workaround a bad design merely to satisfy the
ticket wording.

## Peer Result Protocol

Every Peer returns exactly one primary result state.

### DONE

Include:

- ticket
- summary
- files changed
- tests run
- evidence
- commit
- assumptions discovered

### BLOCKED

Include:

- ticket
- blocker
- evidence
- what is required to continue
- affected dependencies

### CHANGE_REQUEST

Include:

- ticket
- existing assumption or design
- evidence contradicting it
- affected scope
- suggested change
- smallest viable correction
- consequences of not changing it

## Escalation

Peer
→ Lead

Lead escalates to Supervisor when the issue affects:

- multiple ownership scopes
- architecture
- system invariants
- security boundaries
- persistence contracts
- external APIs

Supervisor escalates to Human when the issue changes:

- product semantics
- business requirements
- budget
- delivery scope
- externally committed contracts
- security policy
- irreversible migration decisions

## Evidence Policy

Technical claims that change scope or architecture must include
evidence.

Preferred evidence:

1. failing/passing test
2. reproducible command
3. source code reference
4. runtime log
5. official documentation
6. benchmark
7. reasoned hypothesis

A hypothesis without evidence is not sufficient for an
architecture change.

## Communication Policy

Use sparse communication.

Prefer pointers to:

- spec
- ticket
- WORKSTATE
- ADR
- GLOSSARY
- commit
- test output
- research notes

Do not duplicate large context between agents.

## Language Policy

All Human-facing communication MUST be Vietnamese.

Lead ↔ Supervisor summaries SHOULD be Vietnamese.

Peer technical reports SHOULD be Vietnamese unless repository
conventions require English.

Do not translate:

- code identifiers
- commands
- API names
- exception messages
- log messages
- established library/framework terminology

Source-code comments follow repository conventions.