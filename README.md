# AI Agent Setup

Personal AI agent setup for work, shared for anyone around the world who wants to understand, reuse, or adapt my agent environment.

This repository contains reusable agent skills, workflow guidelines, and conventions that help my AI coding agents plan work, review APIs, write safer migrations, test frontend apps, refactor code, and hand off context between sessions.

> [!NOTE]  
> ☝️ I’m currently looking for a job, particularly with companies in Indonesia that offer on-site or remote positions. I’m also open to fully remote roles with companies based overseas. Here's [my Linkedin profile](https://www.linkedin.com/in/kangabbad/). Thank you 🙏

## What's inside

```txt
.
├── skills/             # Reusable AI agent skills and workflows
├── .skill-lock.json    # Installed skill metadata and sources
├── .gitignore          # Local ignore rules
└── README.md           # Repository overview
```

## Skills catalog

### Planning and process

- `work-planning` — create structured plans for features and refactors.
- `handoff` — compact a session into a clear handoff document.
- `grill-me` — stress-test plans and designs through focused questioning.

### Skill management

- `find-skills` — discover useful installable agent skills.
- `write-a-skill` — create new skills with proper structure and guidance.

### Backend and API

- `design-rest-api` — design and review REST API contracts.
- `backend-endpoint-code-patterns` — standardize Hono endpoint implementation.
- `api-test-collection` — maintain Bruno and Hoppscotch API test collections.

### Database

- `supabase-migrations` — write safe, idempotent Supabase migrations with RLS patterns.

### Frontend testing

- `frontend-testing-strategy` — choose effective testing coverage using testing trophy principles.
- `frontend-e2e-testing` — write stable Playwright end-to-end tests.
- `frontend-test-setup-migration` — set up or migrate Vitest, Testing Library, MSW, and Playwright.

### Code quality and UI patterns

- `refactor-code` — refactor TypeScript, API, Drizzle ORM, and React code consistently.
- `react-table-factory-patterns` — apply table factory and column builder patterns in React routes.

## Managing skills

Useful Skills CLI commands:

```sh
npx skills find [query]
npx skills add <package>
npx skills check
npx skills update
```

Browse more skills at https://skills.sh/.

## Goal

This setup is meant to make AI-assisted development more consistent, reviewable, and practical across real software projects.
