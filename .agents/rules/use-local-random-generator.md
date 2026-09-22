---
title: Keep Deterministic Randomness Local
description: Avoid mutating process-wide random state when generating deterministic mock data.
scope: project
alwaysApply: true
---

# Rule

Do not call the module-level `random.seed()` for request-scoped or reusable mock-data generation. Use a local `random.Random(seed)` instance and pass it through the generation helpers.

# Why

The current generator is called by multiple endpoints and mutates global pseudo-random state. Future concurrent code that also uses `random` can become order-dependent and produce non-reproducible results.

# Repository evidence

- `backend/app/routes.py:73-75` calls `random.seed(seed)` on the global module.
- `backend/app/routes.py:248-259` and `backend/app/routes.py:268-285` regenerate data for separate endpoints using the same seed.
- `backend/app/routes.py:54-65` uses module-level random operations while building each movement.

# Actionable guidance

- Create `rng = random.Random(seed)` inside the top-level generator.
- Pass `rng` to helpers that need random values, and replace module-level calls with methods on that instance.
- Preserve deterministic output for the same seed and add a test that calls generators in different orders.
