---
name: meegle-api-setting
description: |
  Meegle OpenAPI for space/work item settings and configuration. Prerequisites: token and domain — see skill meegle-api-users.
metadata:
  openclaw: {}
  required_credentials:
    domain: "From meegle-api-users"
    plugin_access_token_or_user_access_token: "From meegle-api-users (obtain token first)"
---

# Meegle API — Setting

Setting and configuration related OpenAPIs (e.g. work item types, fields, process templates). Use when you need to read or change space or work item settings.

**Prerequisites:** Obtain domain and access token first; see skill **meegle-api-users** for domain, `plugin_access_token` / `user_access_token`, and request headers.

## Scope

This skill covers setting and configuration operations including:

- Work item creation metadata
- Field definitions
- Process templates and settings
- Space-level configuration

*Detailed API specs will be added as they are documented. Refer to the official Meegle/Lark OpenAPI documentation for current endpoint details.*
