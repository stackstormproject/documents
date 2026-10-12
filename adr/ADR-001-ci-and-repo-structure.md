# ADR-001: CI/CD in infra, co-located tests, lint setup

**Date:** 2026-10-09
**Status:** Accepted
**SRS/SDS Reference:** DevGuide §3.1, §5.2, §7.2

## Context
The guideline says the infra repo holds CI/CD, and that a separate tests repo is needed only if tests are not co-located with the code. The pipeline table lists a Build stage and deploy stages. It names Airbnb + ESLint + Prettier for JavaScript.

## Options Considered
### CI/CD location
- Option A: a full workflow in every code repo. Pros: simple. Cons: duplicated logic, not in infra.
- Option B: reusable workflows in infra, called from backend and frontend with a version tag. Pros: one place to maintain, matches §3.1. Cons: callers depend on infra.

### Tests
- Option C: a separate tests repo. Cons: an extra repo and an extra pipeline.
- Option D: tests inside backend and frontend. Pros: tests change together with the code.

## Decision
Option B (callers pin the infra tag `v1`) and Option D.
The backend has no Build stage: it is plain JavaScript with no compile step. The frontend has one.
Deploy jobs call a hosting deploy hook and skip until the hook secret exists. Production needs a manual approval through a GitHub Environment.
`eslint-config-prettier` is added after the Airbnb config so ESLint and Prettier never disagree. ESLint is pinned to v8 because the Airbnb configs do not support ESLint 9.

## Consequences
**Positive:** one pipeline definition, fewer repos, no style conflicts.
**Negative / Technical debt:** a workflow change needs a PR in infra and a move of the `v1` tag. The hosting service is configured later.