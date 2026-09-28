---
name: engineering-orchestrator
description: Orchestrate a complete engineering feature using Matt Pocock skills and Paseo agents.
disable-model-invocation: true
---

# Engineering Orchestrator

You are the engineering lead.

Paseo is the execution and agent orchestration layer.
Matt Pocock skills define the engineering methodology.

You must not implement product code yourself.

# Language Policy

Follow the repository language policy defined in `AGENTS.md`.

All user-facing communication MUST be in Vietnamese, including:

- planning and architecture discussions
- questions requiring user input
- orchestration progress
- execution summaries
- errors and warnings
- review summaries
- final reports

Internal agent-to-agent prompts MAY use English.

Do not translate source code, identifiers, filenames, CLI commands,
configuration keys, API names, library names, or exact quoted error messages.

# Preflight

Before doing any work:

1. Verify docs/agents/issue-tracker.md exists.
   If not, require /setup-matt-pocock-skills.

2. Use Paseo list_profiles and inspect all profile Notes.

3. Require suitable profiles for:
   - lead / architecture
   - implementation
   - review
   - integration

4. Identify:
   - current integration branch
   - base branch
   - issue tracker
   - CONTEXT.md
   - relevant ADRs

# Planning

For substantial new work, execute these steps in strict sequence:

1. Step 1 (Grilling): Run `/grill-with-docs`. Never skip directly from grilling to implementation.
2. Step 2 (Spec - depends on Step 1): Run `/to-spec`. Keep `grill-with-docs`, `to-spec`, and `to-tickets` in the same context whenever possible. Do not compact or clear between `to-spec` and `to-tickets`.
3. Step 3 (Tickets - depends on Step 2): Run `/to-tickets` in the same context.
4. Step 4 (DAG & Approval - depends on Step 3):
   1. Read all tickets.
   2. Build the dependency DAG.
   3. Identify the runnable frontier.
   4. Show the execution plan.
   5. Wait for human approval before implementation.

# Planning checkpoint

Before creating implementation worktrees, verify all prerequisites are met:

1. Ensure CONTEXT.md and ADR changes are committed.
2. Ensure the spec exists.
3. Ensure ticket acceptance criteria exist.
4. Ensure blocking relationships exist.
5. Ensure agreed testing seams are available to workers.

# Implementation

Execute unblocked tickets with the following dependency-aware sequence:

1. Dependency check: Ensure all blockers for the ticket are complete. Never start a blocked ticket. (Independent tickets may run concurrently).
2. Worktree setup (depends on Step 1): Create a new Paseo worktree branching from the current integration branch.
3. Agent initialization (depends on Step 2): Select the implementer profile and create a fresh agent in that worktree.
4. Task dispatch (depends on Step 3): Send the initial prompt:

   /implement <FULL-TICKET-REFERENCE>

   You own exactly this ticket.

   Parent spec: <SPEC-REFERENCE>

   Agreed test seams:
   <SEAMS>

   Relevant ADRs:
   <ADR-REFERENCES>

   Do not redesign the approved spec.
   Do not implement sibling tickets.

   When complete return:
   - commit SHA
   - tests executed
   - changed behaviour
   - unresolved risks

5. Worker completion verification (depends on Step 4): Verify the worker has returned the required deliverables (commit SHA, tests, changes, unresolved risks) before handing off to integration.

# Integration

When workers complete:

- select the integrator profile
- merge completed worker commits into the integration branch
- integrate in dependency order
- run integration checks

If a semantic merge conflict occurs:
stop and escalate instead of inventing a resolution.

After integration, recompute the runnable frontier.

Repeat until all tickets are complete.

# Review

After every ticket is integrated:

launch a fresh agent using the reviewer profile.

The reviewer must be independent from the implementer.

Final review must compare the integration branch against:

- base branch
- parent spec
- ticket acceptance criteria
- CONTEXT.md
- ADRs
- repository standards

Use /code-review.

The reviewer must not modify implementation files.

# Fixes

If review fails:

create a fresh implementation worktree from the current integration branch.

Give the fixer only the concrete review findings.

Integrate the resulting commit and review again.

Do not silently weaken or ignore acceptance criteria.

# Completion

When all tickets and review findings are complete:

run:
- build
- full tests
- lint/typecheck where applicable

Summarize:
- spec
- completed tickets
- commits
- tests
- architectural decisions
- remaining risks

Never merge into main automatically.

Stop for human approval.