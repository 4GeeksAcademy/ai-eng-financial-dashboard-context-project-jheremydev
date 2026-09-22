---
title: Handle Empty Facet Inputs
description: Make facet construction safe when the movement source returns no records.
scope: project
alwaysApply: true
---

# Rule

Before indexing a sorted movement list, define the response behavior for an empty list. Do not let `build_metrics_facets` fail with an unhandled `IndexError`.

# Why

A filtered, newly initialized, or temporarily unavailable data source can return no movements. The current implementation assumes at least one item and has no explicit empty-state contract.

# Repository evidence

- `backend/app/routes.py:145-155` reads `ordered[0]` and `ordered[-1]` without checking whether `ordered` is empty.
- `backend/app/routes.py:262-265` exposes this function through `/api/metrics/facets`.
- `backend/tests/test_routes.py:85-98` covers only the non-empty generated dataset.

# Actionable guidance

- Choose and document an empty response contract, such as nullable date bounds or a dedicated empty result.
- Implement the guard in `build_metrics_facets` or validate the source before calling it.
- Add a unit test with `[]` and an endpoint test for the resulting HTTP payload.
