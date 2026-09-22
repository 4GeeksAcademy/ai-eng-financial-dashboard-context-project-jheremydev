---
title: Parse Date-Only Values Without Timezone Shifts
description: Preserve the calendar day when grouping backend date values in the frontend.
scope: project
alwaysApply: true
---

# Rule

Do not call `new Date()` directly on a `YYYY-MM-DD` value when the business meaning is a calendar date. Parse the year and month explicitly or use a date-only parser with documented timezone behavior.

# Why

JavaScript can interpret date-only ISO strings as UTC, while `getFullYear()` and `getMonth()` read local time. In some timezones this shifts a value to the previous day or month and changes financial aggregation.

# Repository evidence

- `frontend/src/lib/financial-utils.ts:5-7` groups dates using local `getFullYear()` and `getMonth()`.
- `frontend/src/lib/financial-utils.ts:43` constructs a `Date` directly from `m.create_date`.
- `frontend/src/lib/financial-types.ts:6` documents `create_date` as an ISO date without a time component.
- `backend/app/routes.py:20` models `create_date` as Python `date`.

# Actionable guidance

- For a date-only string, extract `YYYY` and `MM` directly or normalize it through a date-only utility.
- Do not add a timezone conversion without testing dates at month and year boundaries.
- Add tests that run with a non-UTC timezone or otherwise prove that `2024-01-01` remains January 2024.
