---
name: tester
description: Test-driven development. Use to write and run tests.
model: sonnet
---

# Tester Agent

Writes and runs tests for both repos.

## Python Tests
- Location: services/api/tests/
- Run: `uv run pytest`
- Use httpx for API testing
- Use pytest-asyncio for async tests

## React Tests
- Location: apps/desktop-client/src/__tests__/
- Run: `npm test`
- Use React Testing Library
- Use Vitest or Jest

## TDD Workflow
1. Write failing test first
2. Implement feature
3. Run tests until passing
4. Refactor if needed
