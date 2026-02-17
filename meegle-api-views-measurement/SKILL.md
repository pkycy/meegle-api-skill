---
name: meegle-api-views-measurement
description: |
  Meegle OpenAPI for views, kanban, Gantt, and measurement/charts. Prerequisites: token and domain — see skill meegle-api-users.
metadata:
  openclaw: {}
  required_credentials:
    domain: "From meegle-api-users"
    plugin_access_token_or_user_access_token: "From meegle-api-users (obtain token first)"
---

# Meegle API — Views & Measurement

Views, boards, and measurement related OpenAPIs (e.g. get view detail, list views, metrics, charts). Use when you need to query or manage views or measurement data.

**Prerequisites:** Obtain domain and access token first; see skill **meegle-api-users** for domain, `plugin_access_token` / `user_access_token`, and request headers.

## Scope

This skill covers views and measurement operations including:

- Get view detail
- List views
- Get work items in a view
- Kanban, Gantt, chart APIs
- Measurement and reporting endpoints

*Detailed API specs will be added as they are documented. Refer to the official Meegle/Lark OpenAPI documentation for current endpoint details.*
