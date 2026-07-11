---
name: frontend-test-setup-migration
description: Add, standardize, or migrate frontend testing infrastructure in existing JavaScript and TypeScript repos using Vitest, Testing Library, MSW, and Playwright. Use when setting up a new frontend test stack, replacing Jest or brittle legacy patterns, or rolling out testing conventions repo-wide.
---

# Frontend Test Setup And Migration

## Quick Start

Use this skill when repo needs test infrastructure work, not just a few new tests.

## Workflows

### 1. Assess current repo

1. Identify framework, bundler, package manager, CI, and current test stack
2. Find existing test commands, setup files, mocks, helpers, and coverage config
3. Classify state: greenfield setup, partial setup, or migration
4. Note constraints: monorepo, browser env, SSR, legacy snapshots, flaky E2E

### 2. Design target stack

- Default to `Vitest` + Testing Library + `user-event`
- Add `MSW` when UI depends on APIs
- Add `Playwright` only for critical browser journeys
- Keep TypeScript strict mode and lint in validation path
- Match existing conventions unless migration benefit is clear

### 3. Implement minimal viable setup

- Add config files and test scripts
- Create shared `setup` file
- Add one representative passing test
- Add MSW server helpers if network mocking needed
- Keep changes incremental and easy to review

### 4. Migrate safely

- Migrate highest-value tests first
- Replace implementation-detail tests with behavior tests
- Shrink snapshot coverage aggressively
- Remove duplicate helpers and stale mocks
- Keep old and new runners only as long as necessary

### 5. Verify rollout

- Run typecheck
- Run targeted tests, then full test suite if practical
- Confirm CI commands and coverage outputs still work
- Document new commands and conventions

## Rules

- Prefer one canonical setup file per package or app
- Prefer network-boundary mocks over `fetch` stubs
- Prefer semantic queries over test ids
- Avoid introducing two long-term patterns for same layer
- If migration is broad, stage by directory or feature slice

## Deliverables

- Clear migration plan or direct implementation
- Config and setup files with minimal surface area
- Example tests that demonstrate preferred style
- Notes on removed anti-patterns and follow-up cleanup

## References

- Open `REFERENCE.md` for migration checklist, file templates, and rollout guidance
