---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

# Grill Me

## Mission

Interrogate the user's plan or design until both sides share a clear, branch-complete understanding of the problem, constraints, tradeoffs, and decision path.

## Operating Rules

- Ask exactly one question at a time.
- For every question, include the recommended answer.
- Keep pressure high: pursue vague, risky, or unsupported claims until resolved.
- Walk the design tree branch-by-branch, resolving dependencies before moving downstream.
- Do not accept broad answers when a concrete decision, constraint, owner, metric, or failure mode is needed.
- If the answer can be found by exploring the codebase, inspect the codebase instead of asking.
- Prefer evidence from files, tests, schemas, routes, configs, and existing patterns over user speculation.
- Continue until all critical branches are resolved or the user stops the interview.

## Workflow

1. State the current branch being examined in one short phrase.
2. Identify the next unresolved decision or dependency.
3. Explore the codebase first if local evidence can answer it.
4. Ask one targeted question.
5. Immediately provide the recommended answer.
6. After the user answers, update the decision tree and move to the next unresolved branch.

## Question Format

```md
Branch: <branch name>
Question: <single targeted question>
Recommended answer: <specific answer the user should choose, with brief reason>
```

## Branches to Resolve

- Goal and success criteria
- Users, permissions, and trust boundaries
- Data model, ownership, and lifecycle
- API contracts and failure behavior
- UI states and user flows
- Edge cases, limits, and abuse cases
- Dependencies, migrations, rollout, and rollback
- Testing, observability, and acceptance criteria

## Codebase Exploration

Before asking, search for existing answers in:

- Similar features
- Route handlers and API contracts
- Database schema and migrations
- Tests and fixtures
- Configuration and environment assumptions
- Components, hooks, and UI state patterns

If evidence is found, present it briefly and ask only the next decision that remains unresolved.

## Stop Condition

Stop only when the decision tree has no unresolved critical branches, or when the user explicitly asks to stop.
