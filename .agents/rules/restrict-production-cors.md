---
title: Restrict CORS Outside Local Development
description: Keep cross-origin access explicit and environment-specific for deployed backend instances.
scope: project
alwaysApply: true
---

# Rule

Do not deploy the backend with wildcard CORS origins, methods, headers, and credentials enabled. Keep the permissive configuration limited to local development and configure an explicit allowlist for deployed environments.

# Why

Wildcard cross-origin access broadens which browser origins can call the API. Combining it with credentials is especially sensitive and should not become the production default by accident.

# Repository evidence

- `backend/app/main.py:7-13` configures `allow_origins=["*"]`, `allow_credentials=True`, `allow_methods=["*"]`, and `allow_headers=["*"]`.
- `docker-compose.yml:1-22` is the local two-service development setup where this permissive configuration currently runs.

# Actionable guidance

- Read allowed origins from environment-specific configuration.
- Use an explicit frontend origin for staging and production.
- Keep credentials disabled unless the application needs them, and never pair credentials with a wildcard origin.
- Test the CORS policy for an allowed origin and a rejected origin before deployment.
