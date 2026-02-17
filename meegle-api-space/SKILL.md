---
name: meegle-api-space
description: |
  Meegle OpenAPI for space (project) operations. Prerequisites: token and domain — see skill meegle-api-users.
metadata:
  openclaw: {}
  required_credentials:
    domain: "From meegle-api-users"
    plugin_access_token_or_user_access_token: "From meegle-api-users (obtain token first)"
---

# Meegle API — Space

Space (project) related OpenAPIs. Use when you need to manage or query Meegle spaces/projects.

**Prerequisites:** Obtain domain and access token first; see skill **meegle-api-users** for domain, `plugin_access_token` / `user_access_token`, and request headers (`X-Plugin-Token`, `X-User-Key`).

## Scope

This skill covers space (project) operations including:

- List spaces
- Get space details
- Create, update space
- Related space management endpoints

*Detailed API specs (paths, request/response schemas) will be added as they are documented. Refer to the official Meegle/Lark OpenAPI documentation for current endpoint details.*
