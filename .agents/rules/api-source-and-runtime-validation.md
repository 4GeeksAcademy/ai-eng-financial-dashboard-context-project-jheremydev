---
title: Use And Validate The API Data Contract
description: Keep frontend financial data on the API path and validate response shape at runtime.
scope: project
alwaysApply: true
---

# Rule

Use `/api/metrics` as the frontend source of financial movements and validate the response at runtime before computing KPIs or chart data. Do not introduce a second active mock-data source without an explicit migration decision.

# Why

The repository contains a local mock dataset that is not used by the current application, while the dashboard reads from the backend. TypeScript interfaces do not validate JSON received over the network, so a backend contract change can otherwise produce incorrect calculations without a clear failure.

# Repository evidence

- `frontend/src/App.tsx:15-21` fetches financial movements from `/api/metrics`.
- `frontend/src/App.tsx:31-35` passes the decoded JSON directly to `computeKPIs` and `computeMonthlyData`.
- `frontend/src/lib/financial-types.ts:5-12` defines `FinancialMovement` only as a compile-time interface.
- `frontend/src/lib/mock-data.ts:1-12` contains a separate local dataset that is not used by `App`.

# Actionable guidance

- Keep data fetching in the API path unless the application intentionally switches to local fixtures.
- Before calculations, check that the payload is an array and that every movement has valid `create_date`, `amount`, `operation_type`, `category`, and `business_type` values.
- Prefer a schema validator or a dedicated runtime parser over a type assertion.
- If local fixtures are needed for tests or stories, keep them clearly scoped to those consumers and do not silently wire them into production rendering.
- Add tests for malformed payloads and missing fields.
