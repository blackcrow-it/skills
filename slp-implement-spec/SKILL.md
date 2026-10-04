---
name: slp-implement-spec
description: Execute a specification and ticket graph using Supervisor-Lead-Peer governance, Paseo Agent Profiles, evidence-driven investigation, isolated worktrees, scope ownership, integration, and independent review.
disable-model-invocation: true
---

# SLP Implement Spec

Execute an approved specification and ticket graph using
Supervisor-Lead-Peer governance.

The current agent acts as the SLP Lead.

The Lead owns execution orchestration.

The Supervisor owns high-level technical direction,
architecture-level escalation, and communication with the Human.

Peers own bounded technical work.

This skill coordinates execution. It does not replace engineering
discipline skills such as:

- `slp-governance`
- `research`
- `prototype`
- `tdd`
- `diagnosing-bugs`
- `code-review`

---

# 1. Preconditions

Before execution begins:

1. Load `slp-governance`.
2. Read `docs/agents/WORKSTATE.md`.
3. Read `docs/agents/issue-tracker.md`.
4. Read the complete approved specification.
5. Read all tickets and blocking relationships.
6. Read relevant:
   - GLOSSARY
   - ADRs
   - architecture documentation
   - repository instructions
7. Read available Paseo Agent Profiles and their `When to use`
   guidance.
8. Identify Human Constraints already established by the
   Supervisor.
9. Identify current System Invariants.
10. Identify Current Design Decisions.

Do not begin implementation before understanding the complete
dependency graph.

Do not reinterpret Human Constraints as implementation choices.

---

# 2. Execution Authority

The Lead owns:

- WORKSTATE
- ticket graph
- dependency graph
- ready frontier
- investigation state
- mutable scope ownership
- Peer selection
- worktree creation
- execution coordination
- integration
- verification
- review coordination
- execution evidence

The Lead should delegate technical work rather than perform
Peer-owned work directly.

The Lead may create technical Peers.

Peers must not create or coordinate additional agents.

Architecture decisions outside Lead authority must be escalated
to the Supervisor.

Changes to Human Constraints must be escalated through the
Supervisor to the Human.

---

# 3. Initialize WORKSTATE

Before launching any Peer, initialize or refresh WORKSTATE.

Record:

## Goal

The implementation goal derived from the approved specification.

## Human Constraints

Constraints established by the Human through the Supervisor.

These cannot be changed by the Lead or Peers.

## System Invariants

Technical properties that must remain valid.

## Current Design Decisions

Currently accepted technical decisions.

These may be reopened when new evidence justifies reconsideration.

## Ticket Graph

For each ticket record:

- identifier
- goal
- blockers
- status
- owner
- mutable scope
- risk
- evidence

Valid statuses:

- READY
- INVESTIGATING
- RUNNING
- BLOCKED
- CHANGE_REQUEST
- REVIEW
- DONE

## Investigation State

Track Research and Prototype work separately.

For each investigation record:

- ID
- type: RESEARCH or PROTOTYPE
- question
- related ticket
- owner
- status
- evidence
- conclusion
- resulting decision

## Scope Ownership

For mutable production scope record:

- path/module
- owner
- ticket
- acquired time

Invariant:

At any moment each mutable production scope has at most one owner.

---

# 4. Build the Ready Frontier

A ticket is logically READY when all blocking tickets are DONE.

However, logical readiness does not automatically mean it should
be implemented immediately.

For every READY ticket, determine:

1. Are requirements sufficiently understood?
2. Are technical assumptions sufficiently verified?
3. Is feasibility sufficiently established?
4. Is there an unexplained failure involved?
5. Is the implementation scope known?
6. Does mutable scope overlap with another active owner?
7. What level of technical risk exists?

The effective execution frontier is:

logical ready frontier
∩
knowledge ready
∩
feasibility ready
∩
ownership compatible

Do not start production implementation when a material uncertainty
remains unresolved.

---

# 5. Classify the Task

Before creating a Peer, classify the primary uncertainty.

Use the following decision model.

## 5.1 Knowledge Unknown → Research

Use `peer-research` when the main unanswered question is:

"What do we need to know before deciding or implementing?"

Examples:

- unfamiliar repository behaviour
- unclear framework capability
- external API behaviour
- evaluating technical alternatives
- locating an existing architecture pattern
- understanding constraints
- validating documentation assumptions
- determining whether an existing component already solves the need

Research should gather evidence.

Research should not implement the final production solution.

Typical flow:

Research
→ decision
→ Implement

---

## 5.2 Feasibility Unknown → Prototype

Use `peer-prototype` when the main unanswered question is:

"Does this approach actually work under relevant conditions?"

Use when documentation or static reasoning is insufficient.

Examples:

- proof-of-concept
- integration spike
- SDK runtime validation
- concurrency experiment
- protocol experiment
- performance feasibility
- risky library behaviour
- architecture assumption validation

Prototype code is disposable evidence by default.

Do not silently promote prototype code into production code.

Typical flow:

Prototype
→ evidence
→ decision
→ Implement

or:

Research
→ Prototype
→ Implement

---

## 5.3 Implementation Known → Implement

When requirements and implementation direction are sufficiently
established, choose an implementation Peer.

### `peer-implement-fast`

Use when:

- scope is well bounded
- requirements are clear
- approach is established
- technical risk is low or medium

Suitable for:

- CRUD
- endpoints
- handlers
- DTOs
- mappings
- ordinary validation
- straightforward integration
- ordinary tests
- low-risk refactoring

### `peer-implement-quality`

Use when the work affects:

- core domain logic
- security
- authentication
- authorization
- persistence
- transactions
- concurrency
- distributed systems
- public contracts
- cross-module behaviour
- important migrations
- data integrity
- architecture-sensitive implementation

Prefer this profile when correctness is more important than
throughput or cost.

---

# 6. Existing Failure → Debug

Use `peer-debug` when existing behaviour is broken and the root
cause is not established.

Do not send an unexplained failure directly to an implementation
Peer.

Expected flow:

reproduce
→ collect evidence
→ determine root cause
→ define correction
→ implementation task
→ regression test

Use the project's `diagnosing-bugs` skill.

Debugging should establish the cause.

Implementation should apply the correction.

These may be separate Peer assignments when appropriate.

---

# 7. Mechanical Work → Cheap Peer

Use `peer-cheap` only for deterministic, low-risk work.

Examples:

- rename
- formatting
- documentation
- repetitive DTO creation
- mappings
- boilerplate
- simple configuration
- mechanical test expansion
- predictable repetitive migration

Do not use `peer-cheap` for:

- architecture
- security
- concurrency
- destructive migrations
- public contracts
- distributed state
- complex domain behaviour
- ambiguous requirements

If the Peer discovers that significant reasoning is required,
expect CHANGE_REQUEST and reclassify the task.

---

# 8. Research Execution

When Research is required:

1. Create an Investigation entry in WORKSTATE.
2. Set related ticket to `INVESTIGATING` when the ticket cannot
   proceed without the result.
3. Create `peer-research`.
4. Provide:
   - research question
   - ticket pointer
   - spec pointer
   - relevant repository pointers
   - known assumptions
   - evidence requirements
5. Prefer read-only execution.
6. Do not acquire production mutable scope unless absolutely
   required.

Research output must distinguish:

- verified fact
- repository observation
- hypothesis
- recommendation

Expected result:

- DONE
- BLOCKED
- CHANGE_REQUEST

On DONE:

- record evidence
- record conclusion
- update Current Design Decisions if appropriate
- determine whether:
  - implementation can proceed
  - prototype is required
  - Supervisor escalation is required

Research findings do not automatically become architecture
decisions.

---

# 9. Prototype Execution

When Prototype is required:

1. Create an Investigation entry in WORKSTATE.
2. Set related ticket to `INVESTIGATING` when appropriate.
3. Create `peer-prototype`.
4. Define one concrete question the experiment must answer.
5. Define success/failure criteria.
6. Prefer isolated prototype scope.

Prototype work should normally use:

- scratch directory
- temporary branch
- dedicated test harness
- disposable experiment

Avoid modifying production implementation.

Prototype output must report:

- question tested
- experimental setup
- observed result
- limitations
- evidence
- conclusion
- recommendation

Prototype code is not production code by default.

If the experiment succeeds:

→ record evidence
→ decide production implementation

If the experiment fails:

→ reconsider approach
→ Research if needed
→ CHANGE_REQUEST if assumptions are invalid

---

# 10. Mutable Scope Analysis

Before production implementation, identify mutable scope.

Examples:

- `src/Auth/**`
- `src/Billing/**`
- database migration files
- shared interfaces
- package manifests
- deployment configuration

Detect conflicts before parallelization.

Serialize work when tickets need to modify the same shared
surface, especially:

- same database migration
- same public interface
- same central configuration
- same core abstraction
- same generated artifact
- same package manifest
- same schema object

Two tickets may be dependency-independent but still
ownership-conflicting.

Do not parallelize them merely because the ticket graph allows it.

---

# 11. Acquire Ownership

Before launching an implementation Peer:

1. determine mutable production scope
2. verify no conflicting active owner
3. update WORKSTATE
4. set ticket to RUNNING
5. assign owner
6. record acquired scope

Only then create the Peer.

Reading outside owned scope is allowed.

Writing outside owned scope is not allowed unless the Lead:

- transfers ownership
- grants explicit temporary authorization
- creates separate dependent work

---

# 12. Create the Peer

Use the matching Paseo Agent Profile.

Do not select provider or LLM directly when an appropriate
profile exists.

The profile determines execution characteristics.

OmniRoute determines the underlying model/provider route.

The Lead should reason in terms of roles, not raw model names.

---

# 13. Peer Assignment Prompt

Provide pointers instead of copying unnecessary context.

Use this structure:

Goal:
<ticket or investigation goal>

Ticket:
<ticket pointer>

Specification:
<spec pointer>

Owned mutable scope:
<scope or NONE for read-only Research>

Read-only related scope:
<scope>

Human Constraints:
<pointer to WORKSTATE>

System Invariants:
<pointer to WORKSTATE>

Current Design Decisions:
<pointer to WORKSTATE>

Relevant references:
- GLOSSARY
- ADR
- research findings
- prototype findings
- previous commits
- related tests

Instructions:

- Load `slp-governance`.
- Follow the skill appropriate to your role.
- Stay within assigned mutable scope.
- Investigate independently.
- Use repository evidence.
- Do not silently workaround contradictory assumptions.
- Do not create or coordinate other agents.
- Return DONE, BLOCKED or CHANGE_REQUEST.

For implementation:

- use `tdd`

For debugging:

- use `diagnosing-bugs`

For research:

- use `research`

For prototype:

- use `prototype`

For review:

- use `code-review` guidance where applicable

---

# 14. Process Peer Results

Every Peer returns one primary state.

## DONE

### Research DONE

Record:

- findings
- evidence
- conclusion
- confidence/limitations
- resulting decision

Then determine:

Research
→ Implement

or:

Research
→ Prototype

or:

Research
→ Supervisor escalation

### Prototype DONE

Record:

- experiment
- observed behaviour
- evidence
- limitations
- conclusion

Then determine production direction.

Do not merge disposable prototype code unless explicitly promoted.

### Implementation DONE

Verify:

- acceptance criteria
- relevant tests
- scope compliance
- branch state
- integration compatibility

Then:

1. merge safely
2. release ownership
3. set ticket DONE
4. record commit
5. record evidence
6. recompute ready frontier

### Debug DONE

Record root cause and evidence.

If correction remains:

→ create/update implementation work.

Do not confuse root-cause discovery with final implementation.

---

# 15. BLOCKED

On BLOCKED:

Record:

- ticket/investigation
- blocker
- evidence
- affected dependency
- required next action

Determine whether the blocker requires:

- another ticket
- Research
- Prototype
- Debug
- ownership transfer
- Supervisor escalation

Release mutable ownership when continued work cannot proceed.

Do not leave abandoned ownership active.

---

# 16. CHANGE_REQUEST

CHANGE_REQUEST means available evidence contradicts the current
scope, assumption, dependency, or design.

Lead evaluates the evidence.

## Local change

If entirely inside the same ticket and owned scope:

→ update execution details
→ record decision
→ continue work

## Cross-scope change

If another active scope is affected:

→ coordinate with its owner
→ add/update dependency
→ create separate work if necessary

Do not allow the requesting Peer to silently modify another
owner's scope.

## Architecture/System Invariant change

Escalate to `slp-supervisor`.

Provide:

- existing decision
- contradictory evidence
- affected scope
- alternatives
- smallest viable correction
- consequences of keeping current design

Request a decision.

Do not ask Supervisor to perform implementation.

## Human Constraint change

Stop affected execution.

Escalate through Supervisor.

Only the Human may approve changes to Human Constraints.

---

# 17. Supervisor Interaction

Use Supervisor for:

- architecture decisions
- System Invariant changes
- cross-module contract changes
- security boundary changes
- persistence architecture changes
- distributed consistency decisions
- externally visible API redesign
- irreversible technical decisions

Do not escalate:

- variable naming
- ordinary refactoring
- local implementation choices
- private helper structure
- test organization
- formatting
- routine library usage

The Lead owns execution.

The Supervisor owns high-level technical direction.

---

# 18. Recompute the Frontier

After every:

- Research completion
- Prototype completion
- ticket completion
- blocker resolution
- CHANGE_REQUEST decision
- integration event

recompute:

1. logical dependencies
2. unresolved knowledge uncertainty
3. unresolved feasibility uncertainty
4. ownership conflicts
5. risk classification

Then identify the new effective ready frontier.

---

# 19. Parallelism Policy

Prefer useful parallelism over maximum parallelism.

Parallelize when:

- dependencies permit it
- mutable scopes do not overlap
- one task does not require evidence from another
- architecture assumptions are sufficiently stable

Good parallelization:

Research A
Research B
Independent implementation scopes
Independent reviews

Bad parallelization:

Prototype and production implementation of the same unresolved idea

Two migrations modifying the same schema object

Two Peers modifying the same public interface

Implementation depending on unresolved architecture Research

Do not launch agents merely because capacity is available.

---

# 20. Integration Verification

After implementation tickets are DONE, run appropriate repository
verification.

Examples:

- build
- compile
- typecheck
- lint
- focused tests
- integration tests
- architecture tests
- full relevant test suite

Record:

- command
- result
- timestamp if useful
- relevant failure/passing evidence

Do not proceed to final review with unexplained verification
failures.

---

# 21. Independent Review

After integration verification, perform independent review.

Use:

- `peer-review-gpt`
- `peer-review-gemini`

Prefer at least one reviewer from a model family different from
the dominant implementation family.

Run two independent review axes where practical:

## Standards Review

Review against:

- repository conventions
- architecture rules
- maintainability
- safety
- quality
- tests

## Spec Review

Review against:

- specification
- acceptance criteria
- user-visible behaviour
- missing requirements
- regression risk

Reviewers should normally be read-only.

Do not allow reviewers to silently fix findings.

---

# 22. Process Review Findings

Classify findings:

- confirmed defect
- probable risk
- optional improvement
- stylistic preference

Accept findings based on evidence.

For accepted implementation changes:

→ create a fix task
→ assign appropriate implementation Peer
→ use TDD for behaviour changes
→ verify again

If review reveals architecture contradiction:

→ CHANGE_REQUEST
→ Supervisor

Do not expand feature scope simply because a reviewer suggests
unrelated improvements.

---

# 23. Final Completion Conditions

The workstream is complete only when:

- all required tickets are DONE
- all blocking investigations are resolved
- no unresolved BLOCKED state remains
- no unresolved CHANGE_REQUEST remains
- no active mutable ownership remains
- integration verification passes
- required review findings are resolved
- Human Constraints remain satisfied

Update WORKSTATE with final execution status.

Record:

- integrated commits
- final verification evidence
- review result
- remaining non-blocking recommendations

If repository workflow requires a PR, continue with `/pr`.

After merge/completion, use `/retro` when useful to improve:

- profile selection
- routing rules
- ticket quality
- skill instructions
- orchestration policy
- model allocation