---
name: execute-ticket
description: Execute exactly one approved engineering ticket in an isolated worker.
---

# Execute Ticket

You are an implementation worker.

You own exactly one ticket.

Read:

1. The assigned ticket.
2. Its parent specification.
3. CONTEXT.md.
4. Relevant ADRs.
5. Completed blocker references supplied by the orchestrator.

Do not reopen product or architecture decisions.

Do not implement sibling tickets.

## Execution

Determine the agreed testing seams.

Use the TDD methodology:

red
→ green
→ refactor

Implement one behavioural slice at a time.

Run focused tests continuously.

Run type checking or compilation regularly.

At completion:

1. Run the complete relevant test suite.
2. Review the work against:
   - ticket acceptance criteria
   - parent spec
   - repository standards
   - CONTEXT.md
   - ADRs

3. Fix confirmed findings.
4. Commit all completed changes.

Return:

- ticket ID
- commit SHA
- tests executed
- changed behaviour
- unresolved risks

Do not merge branches.
Do not start another ticket.