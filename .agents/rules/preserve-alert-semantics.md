---
title: Preserve Alert Baseline Semantics
description: Keep the meaning of outcome alert thresholds explicit when changing alert calculations.
scope: project
alwaysApply: true
---

# Rule

Treat the current outcome-alert baseline as a cumulative average of all preceding periods. Do not change it to a previous-period or rolling-window comparison without renaming, documenting, and testing the changed behavior.

# Why

The threshold value alone does not reveal which baseline is used. Changing the baseline changes which financial periods are flagged even if the endpoint shape remains identical.

# Repository evidence

- `backend/app/routes.py:207-224` accumulates historical outcomes and computes the baseline from all prior values.
- `backend/app/routes.py:342-360` exposes the behavior through `/api/metrics/alerts`.
- `backend/tests/test_routes.py:167-180` checks the response shape but does not define the baseline semantics numerically.

# Actionable guidance

- Keep the cumulative-average behavior unless the product requirement explicitly changes.
- If changing it, update the endpoint or parameter name to make the baseline clear.
- Add a test with at least three periods whose cumulative and previous-period baselines produce different alert results.
