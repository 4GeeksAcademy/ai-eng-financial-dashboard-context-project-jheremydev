---
title: Derive Display Periods From Data
description: Keep dashboard period labels consistent with the dates returned by the backend.
scope: project
alwaysApply: true
---

# Rule

Do not hardcode a year, date range, or reporting period in a frontend label when that value can be derived from the data being displayed.

# Why

The backend generates movement dates relative to the current date, so a static label can describe a different period than the metrics and charts actually show.

# Repository evidence

- `frontend/src/App.tsx:53` passes the fixed label `2024 - Full Year` to `DashboardHeader`.
- `backend/app/routes.py:54-59` derives movement years from `date.today()`.

# Actionable guidance

- Derive the visible period from the response data or from one shared period configuration.
- When changing backend date generation, update the period derivation and its tests in the same change.
- Add a test that fails when the displayed period and the returned data year diverge.
