---
title: Test Boundary And Contract Cases
description: Extend focused tests to cover empty inputs, date boundaries, and externally visible contracts.
scope: project
alwaysApply: true
---

# Rule

For new or changed data-processing behavior, test the happy path and the boundary cases that can change the result: empty collections, date and timezone boundaries, malformed external data, and semantic threshold differences.

# Why

The existing tests establish common endpoint behavior but do not protect the most fragile assumptions identified in the repository.

# Repository evidence

- `backend/tests/test_routes.py:85-98` tests facets only with the non-empty generated dataset.
- `frontend/src/lib/financial-utils.test.ts:45-71` tests cross-year ordering but not timezone-sensitive date-only parsing.
- `backend/tests/test_routes.py:167-180` verifies alert response fields but not the cumulative-baseline meaning.
- `frontend/src/lib/financial-utils.test.ts:25-43` demonstrates the project's existing focused unit-test style.

# Actionable guidance

- Add the smallest focused test beside the changed function or endpoint.
- For date grouping, include the first day of a month, the last day of a month, and a non-UTC timezone case where relevant.
- For collections, include an empty input and assert the intended result or error contract.
- For API consumers, include invalid or incomplete payloads and assert that they fail clearly.
- Do not replace focused tests with only a broad end-to-end check.
