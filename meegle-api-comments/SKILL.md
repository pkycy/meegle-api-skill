---
name: meegle-api-comments
description: |
  Meegle OpenAPI for comments on work items or other entities. Prerequisites: token and domain — see skill meegle-api-users.
metadata:
  openclaw: {}
  required_credentials:
    domain: "From meegle-api-users"
    plugin_access_token_or_user_access_token: "From meegle-api-users (obtain token first)"
---

# Meegle API — Comments

Comment-related OpenAPIs (e.g. add, list, update comments on work items). Use when you need to create or query comments.

**Prerequisites:** Obtain domain and access token first; see skill **meegle-api-users** for domain, `plugin_access_token` / `user_access_token`, and request headers.

## Scope

This skill covers comment operations including:

- Create comment on work items or other entities
- List comments
- Update and delete comments
- Related comment endpoints

*Detailed API specs will be added as they are documented. Refer to the official Meegle/Lark OpenAPI documentation for current endpoint details.*
