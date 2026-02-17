---
name: meegle-api
description: |
  Meegle Open API skills (index). Read the specific skill for your need. Order: Credentials, Users, Space, Work Items, Setting, Comments, Views & Measurement.
metadata:
  openclaw: {}
  required_credentials:
    plugin_id: "Plugin ID; see meegle-api-credentials"
    plugin_secret: "Plugin secret; see meegle-api-credentials"
    domain: "API host; see meegle-api-credentials"
  optional_context:
    project_key: "Space identifier; in Meegle Developer Platform double-click the project icon to get it"
    user_key: "User identifier; in Meegle Developer Platform double-click the avatar to get it"
    user_access_token: "Required for write operations on behalf of user; obtain via OAuth (see meegle-api-users)"
---

# Meegle API (index)

Meegle OpenAPI is split into the following skills. Read **meegle-api-credentials** first for domain and token; then read the skill that matches your task.

| Order | Sub-skill (path) | When to read |
|-------|------------------|--------------|
| 1 | **meegle-api-credentials/SKILL.md** | Obtain domain and access token (plugin/user). Read this before any other Meegle API call. |
| 2 | **meegle-api-users/SKILL.md** | Context (project_key, user_key), request headers, global constraints, user-related OpenAPIs. |
| 3 | **meegle-api-space/SKILL.md** | Space (project) operations. |
| 4 | **meegle-api-work-items/SKILL.md** | Create, get, update work items (tasks, stories, bugs). |
| 5 | **meegle-api-setting/SKILL.md** | Settings, work item types, fields, process configuration. |
| 6 | **meegle-api-comments/SKILL.md** | Comments on work items or other entities. |
| 7 | **meegle-api-views-measurement/SKILL.md** | Views, kanban, Gantt, charts, measurement. |

Each sub-skill lives under `meegle-api-skill/` (e.g. `meegle-api-users/SKILL.md`). Use the `Read` tool on the relevant path when you need that API area.
