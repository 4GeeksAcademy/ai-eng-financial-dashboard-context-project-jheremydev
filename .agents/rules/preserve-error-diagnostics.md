---
title: Preserve Technical Error Diagnostics
description: Show safe user-facing errors without discarding information needed to debug failures.
scope: project
alwaysApply: true
---

# Rule

When catching frontend request errors, keep the technical error available to developers through structured logging or an equivalent diagnostic channel while showing a safe, user-oriented message in the UI.

# Why

The current flow creates an error containing the HTTP status and then ignores it in the catch handler. Failures remain understandable to users but lose the detail needed to diagnose API or proxy problems.

# Repository evidence

- `frontend/src/App.tsx:16-20` creates an error containing the response status.
- `frontend/src/App.tsx:31-36` catches the error without binding or logging it.
- `frontend/src/App.tsx:37-40` renders a generic recovery message.

# Actionable guidance

- Bind the caught error and log a structured message with the endpoint and status when available.
- Keep sensitive response bodies and credentials out of logs.
- Preserve the generic user-facing message unless the application has a safe error taxonomy.
- Add a test or documented verification path for non-2xx responses and network failures.
