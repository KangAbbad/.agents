---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up.
argument-hint: "What will the next session be used for?"
disable-model-invocation: true
---

# Handoff

Create a concise handoff document so a fresh agent can continue the work.

## Workflow

1. Identify the current workspace.
   - If working in a specific project, save handoffs under `docs/handoff/{date}/{title}.md`.
   - `{date}` must use `YYYYMMDD` format.
   - `{title}` must be a short kebab-case topic.
   - If no specific workspace applies, save under a temp directory using the same `{date}/{title}.md` pattern.
2. If arguments are provided, treat them as the next-session focus and tailor the handoff around that use case.
3. Review current conversation and capture only information needed to continue.
4. Redact secrets, credentials, tokens, personal data, and sensitive environment details.
5. Do not duplicate content already captured in artifacts such as PRDs, plans, ADRs, issues, commits, or diffs. Reference those artifacts by path or URL instead.
6. Write the handoff document.

## Document structure

Use this structure unless a shorter form is sufficient:

```md
# Handoff: <topic>

## Next-session focus

<arguments or inferred focus>

## Current state

<what has been done and where things stand>

## Key context

<decisions, constraints, conventions, important files>

## Artifacts to read

- `<path-or-url>` — <why it matters>

## Open tasks

- <next action>

## Suggested skills

- `<skill-name>` — <why it may help>

## Validation status

<checks run, results, blockers>

## Sensitive information

<state that sensitive information was redacted, or note none was present>
```

## Suggested skills section

Include skills that the next agent should consider loading based on remaining work. Prefer installed skill names when known. Keep reasons short and task-specific.

## Output

After writing the document, report the file path and any validation performed.
