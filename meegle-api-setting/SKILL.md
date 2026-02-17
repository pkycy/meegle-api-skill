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

- Space Setting: Get work item types in space, Get business line details in space, Work item settings
- Roles, Workflow settings
- (Comments, Views & Measurement, Webhooks, Event capability — see respective skills)

---

## Get Work Item Types in Space

Obtain the list of all work item types in the space and their docking identifiers (**work_item_type_key** / **api_name**) for use in subsequent APIs. Permission: Permission Management > Configuration Categories.

### When to Use

- When you need **work_item_type_key** or **api_name** for other interfaces (work item CRUD, workflow, lists, etc.)
- When building space configuration UIs or listing available work item types (story, issue, version, sprint, chart, sub_task, etc.)
- When checking **is_disable** or **enable_model_resource_lib** per type

### API Spec: get_work_item_types_in_space

```yaml
name: get_work_item_types_in_space
type: api
description: >
  Obtain all work item types in the space (WorkItemKeyType). Returns type_key, name,
  api_name, is_disable, enable_model_resource_lib for each type.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/work_item/all-types
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space unique identifier (project_key). Double-click space name in Meegle project space to obtain.

outputs:
  data:
    type: array
    description: >
      List of work item types per WorkItemKeyType: type_key, name, is_disable,
      api_name, enable_model_resource_lib. type_key / api_name are used as work_item_type_key in other APIs.

constraints:
  - Permission: Permission Management – Configuration Categories

error_mapping:
  10023: User not exist (X-User-Key in header incorrect or user not found)
```

### Usage notes

- **type_key** and **api_name**: Use as **work_item_type_key** in work item, workflow, list, and other APIs (e.g. `story`, `issue`, `version`, `sub_task`).
- **is_disable**: Indicates whether the type is disabled (value meaning per product; typically 2 = enabled).
- **enable_model_resource_lib**: Whether the type supports model resource library.
- **X-User-Key**: Must be a valid user key; invalid or missing key returns 10023.

---

## Get Business Line Details in Space

Obtain the business line information of the space. Response follows the Business structure (tree with id, name, role_owners, watchers, level_id, parent, children, etc.). For the platform feature, see Business Line Configuration. Permission: Permission Management – Configuration.

### When to Use

- When building business-line UIs or selectors that need the full business line tree for the space
- When you need role_owners, watchers, super_masters, or hierarchy (parent, children, level_id) per business line
- When validating or displaying business line IDs (id) for work items or filters

### API Spec: get_business_line_details_in_space

```yaml
name: get_business_line_details_in_space
type: api
description: >
  Obtain business line information of the space. Returns list of Business objects
  (tree: id, name, role_owners, watchers, level_id, parent, children, disabled, labels, order, project, super_masters).

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/business/all
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space unique identifier (project_key). Double-click space name in Meegle project space to obtain.

outputs:
  data:
    type: array
    description: >
      List of Business objects. Each has id, name, role_owners (role, owners, name),
      watchers, level_id, parent, disabled, labels, order, project, super_masters,
      children (nested Business objects with same structure).

constraints:
  - Permission: Permission Management – Configuration

error_mapping:
  1000052062: Project key is wrong (not found simple name; project_key incorrect)
```

### Usage notes

- **data**: Root-level array may represent top-level business lines; each item can have **children** forming a tree. **parent** links to parent id; **level_id** indicates depth.
- **role_owners**: Can be array of `{ role, owners }` or object keyed by role (e.g. `role_test: { name, owners, role }`); **owners** are user_key lists.
- **watchers** / **super_masters**: user_key arrays. **project** is the space/project identifier.
- For business line configuration in the product, refer to **Business Line Configuration** in the platform docs.

---

## Get Basic Work Item Settings

Obtain the basic information configuration of the specified work item type (type_key, name, flow_mode, schedule/estimate/actual-work-time fields, belong_roles, resource library settings, etc.). Permission: Permission Management – Work Item Instance. For details, see Permission Management.

### When to Use

- When building work item type configuration UIs or displaying current settings before editing (pair with Update Work Item Basic Information Settings)
- When you need flow_mode (workflow vs stateflow), enable_schedule, field keys/names, belong_roles, or resource_lib_setting
- When checking actual_work_time_switch or enable_model_resource_lib for a type

### API Spec: get_basic_work_item_settings

```yaml
name: get_basic_work_item_settings
type: api
description: >
  Obtain basic information configuration of the specified work item type:
  type_key, name, flow_mode, api_name, description, is_disabled, is_pinned,
  enable_schedule, schedule/estimate/actual_work_time field keys and names,
  belong_roles, actual_work_time_switch, enable_model_resource_lib, resource_lib_setting.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/work_item/type/{work_item_type_key}
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space; must match the type to query.

outputs:
  data:
    type: object
    description: >
      type_key, name, flow_mode (workflow | stateflow), api_name, description,
      is_disabled, is_pinned, enable_schedule, schedule_field_key/name,
      estimate_point_field_key/name, actual_work_time_field_key/name,
      belong_roles (id, name, key), actual_work_time_switch, enable_model_resource_lib,
      resource_lib_setting (enable_roles, enable_fields with field_alias, field_key, field_name, field_type_key).

constraints:
  - Permission: Permission Management – Work Item Instance

error_mapping:
  30014: Work item type not found (no type for given work_item_type_key)
```

### Usage notes

- **flow_mode**: **workflow** = node flow; **stateflow** = state flow. **enable_schedule** applies to overall scheduling; when true, schedule/estimate/actual field settings take effect; when false, only supported for state-flow work items.
- **belong_roles**: Array of `{ id, name, key }` for roles associated with scheduling and score estimation. **resource_lib_setting**: **enable_roles** and **enable_fields** describe resource library configuration when **enable_model_resource_lib** is true.
- **actual_work_time_switch**: true = work item time managed via Open API; false = via platform standard functions.

---

## Update Work Item Basic Information Settings

Update the basic information configuration of the specified work item type (description, disabled/pinned, schedule and estimate fields, role keys, actual work time switch). Permission: Permission Management – Work Items. For permission details, see Permission Management.

### When to Use

- When changing work item type settings: description, is_disabled, is_pinned, schedule/estimate/actual-work-time fields, belong roles, or actual_work_time_switch
- When configuring which fields are used for scheduling, estimate points, and actual work time
- When enabling or disabling the API for registering work-item working hours

### API Spec: update_work_item_basic_information_settings

```yaml
name: update_work_item_basic_information_settings
type: api
description: >
  Update basic information configuration of the specified work item type:
  description, is_disabled, is_pinned, enable_schedule, field keys, belong_role_keys, actual_work_time_switch.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: PUT
  url: https://{domain}/open_api/{project_key}/work_item/type/{work_item_type_key}
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space; must match the type to update.

inputs:
  description:
    type: string
    required: false
    description: Work item type description.
  is_disabled:
    type: boolean
    required: false
    description: true = disable this work item type; false = enabled.
  is_pinned:
    type: boolean
    required: false
    description: true = show as entry in navigation bar; false = do not show.
  enable_schedule:
    type: boolean
    required: false
    description: true = work item supports overall scheduling; false = does not.
  schedule_field_key:
    type: string
    required: false
    description: Field key for scheduling; use a date-range type field.
  estimate_point_field_key:
    type: string
    required: false
    description: Field key for estimated score; use a numeric field.
  actual_work_time_field_key:
    type: string
    required: false
    description: Field key for actual working hours; use a numeric field.
  belong_role_keys:
    type: array
    items: string
    required: false
    description: >
      Keys of associated roles for scheduling and score estimation (score split among roles).
      Roles must already exist in process role configuration.
  actual_work_time_switch:
    type: boolean
    required: false
    description: Whether to enable the API for registering work-item working hours.

outputs:
  data:
    type: object
    description: Empty object on success.

constraints:
  - Permission: Permission Management – Work Items
  - Current user must have space admin permission (else 10005)

error_mapping:
  10005: No project admin permission (current user is not space administrator)
  30014: Work item not found (work_item_type_key incorrect)
```

### Usage notes

- **work_item_type_key**: From **Get Work Item Types in Space** (e.g. `story`, `issue`). Wrong or non-existent type returns 30014.
- **belong_role_keys**: Parameter name in API is **belong_role_keys** (some examples may show **belong_roles_keys**). Roles must exist in process role configuration.
- **schedule_field_key** / **estimate_point_field_key** / **actual_work_time_field_key**: Use field keys from field configuration (e.g. Get Field Information). Date-range for schedule; numeric for estimate and actual work time.
- Space administrator permission is required (10005 if not admin).

---

## Get Field Information

Obtain the basic information of all fields under the specified space or, when **work_item_type_key** is provided, under that work item type. Response follows the SimpleField structure (field_key, field_type_key, options for select, compound_fields for compound_field). Permission: Permission Management – Configuration.

### When to Use

- When building field selectors or form UIs that need field keys, types, aliases, and scopes (work_item_scopes)
- When you need option lists for select-type fields (label, value, order, is_disabled, etc.) or sub-fields for compound_field
- When resolving field_key for Update Work Item Basic Information Settings (schedule_field_key, estimate_point_field_key, actual_work_time_field_key) or for work item read/write APIs

### API Spec: get_field_information

```yaml
name: get_field_information
type: api
description: >
  Obtain all fields under the space or under a work item type. Returns SimpleField
  list: field_key, field_type_key, field_alias, field_name, is_custom_field,
  work_item_scopes, value_generate_mode, relation_id; options for select; compound_fields for compound_field.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/{project_key}/field/all
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).

inputs:
  work_item_type_key:
    type: string
    required: false
    description: Work item type. Obtain via Get work item types in space. Omit to get all fields in space.

outputs:
  data:
    type: array
    description: >
      List of SimpleField. Each has field_key, field_type_key, field_alias, field_name,
      is_custom_field, work_item_scopes, value_generate_mode, relation_id. Select-type
      fields include options (order, color, is_visibility, is_disabled, label, value,
      work_item_type_key). compound_field type includes compound_fields (array of
      field objects with field_key, field_type_key, field_name, etc.).

constraints:
  - Permission: Permission Management – Configuration

error_mapping:
  30001: Data not found (project_key does not match work_item_type_key; no such work item type in this space)
```

### Usage notes

- **work_item_type_key**: Omit to return all fields in the space; set to a type (e.g. `story`, `chart`) to return only fields scoped to that type. If the type does not exist in the space, 30001 is returned.
- **data** items: Use **field_key** when updating work item type settings (schedule_field_key, etc.) or when sending field values in work item APIs. **field_type_key** (e.g. `multi_text`, `select`, `compound_field`) determines value shape.
- **options**: For **select** (and similar) fields; **value** is what to send in API payloads; **label** is display. **compound_fields**: For **compound_field** type; each sub-field has its own field_key and field_type_key.

---

## Create Custom Field

Create a new custom field under the specified work item type. Returns the new **field_key**. Permission: Permission Management – Work Item Instances. For feature details, see Permission Management.

### Points to note

- Work item relationship field default data visibility is fixed; conditional data range cannot be modified.
- Default values are not supported for attachment type or for external system signal / multi-value external system signal.
- Modifying field validity (default valid) is not supported.
- Updating work item type, creator, submission time, completion time, and business line fields is not supported.
- Voting fields are not supported.

### When to Use

- When adding a new custom field (text, select, date, user, number, link, etc.) to a work item type
- When reusing options from another field (value_type 1 + reference_work_item_type_key, reference_field_key) or defining custom options (field_value)
- When configuring team_option for cascading single/multi-select (tree_select, tree_multi_select); mutually exclusive with field_value

### API Spec: create_custom_field

```yaml
name: create_custom_field
type: api
description: >
  Create a custom field under the specified work item type. Returns field_key.
  Supports text, select, tree_select, date, user, number, link, multi_file, bool,
  work_item_related_select, etc. field_value and team_option are mutually exclusive.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/{project_key}/field/{work_item_type_key}/create
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

inputs:
  field_name:
    type: string
    required: true
    description: Field name; must not duplicate other fields.
  field_alias:
    type: string
    required: false
    description: Field alias; cannot duplicate other fields of the same work item type.
  field_type_key:
    type: string
    required: true
    description: >
      Field type. See Attributes and fields. Supported: text, multi_text, select,
      multi_select, tree_select, tree_multi_select, radio, user, multi_user, date,
      schedule, link, number, multi_file, bool, signal, work_item_related_select,
      work_item_related_multi_select.
  value_type:
    type: integer
    required: false
    description: >
      Option source for select/multi_select/tree_select/tree_multi_select/radio.
      0: Custom (default); 1: Reuse. When 1, reference_work_item_type_key and reference_field_key required.
  field_value:
    type: object
    required: false
    description: >
      Option values; structure per Attributes and fields. For select, multi_select,
      tree_select, tree_multi_select, radio. Mutually exclusive with team_option.
  reference_work_item_type_key:
    type: string
    required: false
    description: Required when value_type = 1.
  reference_field_key:
    type: string
    required: false
    description: Required when value_type = 1.
  is_multi:
    type: boolean
    required: false
    description: For text: true = multi-line; false = single-line (default).
  format:
    type: boolean
    required: false
    description: For date: true = date only; false = date + time (default).
  free_add:
    type: integer
    required: false
    description: For select/tree_select etc.: 1 = allow users to add options; 2 = No (default).
  work_item_relation_uuid:
    type: string
    required: false
    description: Work item relationship ID from Get field information. Required for work_item_related_select / work_item_related_multi_select.
  default_value:
    type: object
    required: false
    description: >
      Default value; structure per field type (Attributes and fields). Not supported
      for attachment, external system signal, multi-value external system signal.
  help_description:
    type: string
    required: false
    description: Help instructions.
  authorized_roles:
    type: array
    items: string
    required: false
    description: >
      Roles or system keys that can access/operate this field. Default "Anyone".
      e.g. _master (Administrator), _owner (Creator), _role (Requirement-related person).
  team_option:
    type: object
    required: false
    description: >
      Team scope per TeamOption. Only for tree_select, tree_multi_select.
      Mutually exclusive with field_value.

outputs:
  data:
    type: string
    description: Created field key (e.g. field_73jds7).

constraints:
  - Permission: Permission Management – Work Item Instances
  - field_value and team_option cannot both be sent

error_mapping:
  1000051468: Field alias repeated (field_alias duplicate)
  1000051469: Invalid update version (field being concurrently updated)
  1000051750: Field name has been used (field_name duplicate)
  1000053603: Field name length exceeded (max characters for field name)
```

### Usage notes

- **field_type_key**: Use supported API types (e.g. **text**, **select**, **date**, **user**, **number**, **link**, **multi_file**, **bool**, **work_item_related_select**, **work_item_related_multi_select**). See **Attributes and fields** and the field type support list in the product docs.
- **field_value** vs **team_option**: Use **field_value** for custom/reused option values for select/tree_select etc.; use **team_option** only for cascading single/multi-select with team scope. Do not send both.
- **value_type 1**: Set **reference_work_item_type_key** and **reference_field_key** to reuse options from another field.
- **work_item_relation_uuid**: From Get Field Information (relation or work item relation fields). Required for **work_item_related_select** / **work_item_related_multi_select**.
- Default values are not supported for attachment, external system signal, or multi-value external system signal fields.

---

## Update Custom Field

Update the configuration of the specified custom field. Identify the field by **field_key** in the body. Permission: Permission Management – Configuration. For details, see Permission Management.

### Points to note

- Work item relationship field default data visibility is fixed; conditional data range cannot be modified.
- Default values are not supported for attachment type or for external system signal / multi-value external system signal.
- Modifying field validity (default valid) is not supported.
- Updating work item type, creator, submission time, completion time, and business line fields is not supported.
- Voting fields are not supported.
- When modifying or adding sub-options (cascading options), **parent_value** is required.

### When to Use

- When changing field name, alias, help description, default value, authorized roles, or option list (field_value with action add/modify/delete)
- When updating team scope (team_option) for cascading single/multi-select; mutually exclusive with field_value
- When updating a composite field: pass parent field_key to change only field_name, field_alias, help_description; pass child field_key to update all attributes

### API Spec: update_custom_field

```yaml
name: update_custom_field
type: api
description: >
  Update configuration of the specified custom field. field_key in body identifies
  the field. field_value supports option actions (add/modify/delete); parent_value
  required for sub-options. field_value and team_option are mutually exclusive.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: PUT
  url: https://{domain}/open_api/{project_key}/field/{work_item_type_key}
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

inputs:
  field_key:
    type: string
    required: false
    description: >
      Unique identifier of the field from Get Field Information. For composite
      fields: parent field_key allows only field_name, field_alias, help_description;
      child field_key allows all attributes.
  field_name:
    type: string
    required: false
    description: Field name.
  field_value:
    type: array
    required: false
    description: >
      Option updates. Each item: label, value (option id), action (0 add, 1 modify, 2 delete).
      For sub-options include parent_value. Structure per Attributes and fields.
      Mutually exclusive with team_option.
  free_add:
    type: integer
    required: false
    description: 1 = allow users to add options; 2 = No (default). For select/tree_select etc.
  work_item_relation_uuid:
    type: string
    required: false
    description: Work item relationship ID from Get Field Information. For work_item_related_select / work_item_related_multi_select.
  default_value:
    type: object
    required: false
    description: Default value; structure per field type. Not supported for attachment, signal, multi-value signal.
  field_alias:
    type: string
    required: false
    description: Field alias; cannot duplicate other fields of same work item type.
  help_description:
    type: string
    required: false
    description: Help instructions.
  authorized_roles:
    type: array
    items: string
    required: false
    description: Role or system keys (e.g. _master, _owner, _role). Default "Anyone".
  team_option:
    type: object
    required: false
    description: Team scope per TeamOption. Only for tree_select, tree_multi_select. Mutually exclusive with field_value.

outputs:
  data:
    type: object
    description: Empty on success (no data in response).

constraints:
  - Permission: Permission Management – Configuration
  - field_value and team_option cannot both be sent

error_mapping:
  1000051468: Field alias repeated (replace field_alias)
  1000051750: Field name has been used (replace field_name)
  1000050746: Field type not supported
  1000053603: Field name length exceeded (max 255 characters)
  1000053604: Field description length exceeded (max 255 characters)
  1000053605: Option count exceeded (up to {Number} options)
```

### Usage notes

- **field_key**: Required to identify which field to update; from **Get Field Information**. For **compound_field**: use parent **field_key** to change only name/alias/help_description; use child **field_key** to update all attributes.
- **field_value** option actions: **action** 0 = add option, 1 = modify option, 2 = delete option. Include **parent_value** when adding or modifying sub-options (cascading).
- **field_value** vs **team_option**: Do not send both. Use **field_value** for option list changes; use **team_option** for team scope on tree_select / tree_multi_select.
- Default values are not supported for attachment, external system signal, or multi-value external system signal.

---

## Get the List of Work Item Relationships

Obtain the list of work item association relationships under the specified space. Response follows the WorkItemRelation structure (id, name, relation_type, work_item_type_key/name, relation_details). Permission: Permission Management – Work Item Instances.

### When to Use

- When building relationship configuration UIs or listing which work item types are linked (e.g. story → sprint)
- When you need relationship **id** or **relation_type** for Create/Update/Delete Work Item Relationships or for work_item_relation_uuid in custom fields
- When displaying **relation_details** (project_key, project_name, work_item_type_key, work_item_type_name) per relation

### API Spec: get_list_of_work_item_relationships

```yaml
name: get_list_of_work_item_relationships
type: api
description: >
  Obtain the list of work item association relationships under the space.
  Returns WorkItemRelation list: id, name, disabled, relation_type, work_item_type_key/name, relation_details.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/work_item/relation
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).

outputs:
  data:
    type: array
    description: >
      List of WorkItemRelation. Each has id, name, disabled, relation_type,
      work_item_type_key, work_item_type_name, relation_details (array of
      project_key, project_name, work_item_type_key, work_item_type_name).

constraints:
  - Permission: Permission Management – Work Item Instances

error_mapping:
  1000052062: Project key is wrong (project_key incorrect; provide correct value)
```

### Usage notes

- **data** items: Use **id** when creating/updating/deleting work item relationships or when referencing in custom field **work_item_relation_uuid**. **relation_type** and **relation_details** describe the link (source type and target project/type per detail).
- **relation_details**: Each entry gives the target space (**project_key**, **project_name**) and target work item type (**work_item_type_key**, **work_item_type_name**) for that relation.

---

## Create Work Item Relationships

Add a work item association relationship under the specified space. Returns the new relationship **relation_id** (UUID). Permission: Permission Management – Work Item Instances. For details, see Permission Management.

### When to Use

- When creating a new work item relationship (e.g. story → story in same or another space)
- When defining **relation_details** (target project_key and work_item_type_key per link)
- When you need the returned **relation_id** for custom fields (work_item_relation_uuid) or for Update/Delete Work Item Relationships

### API Spec: create_work_item_relationships

```yaml
name: create_work_item_relationships
type: api
description: >
  Add a work item association relationship under the space. Returns relation_id (UUID).
  Requires project_key, work_item_type_key (source type), name, and relation_details.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/work_item/relation/create
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params: {}

inputs:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type (source). Obtain via Get work item types in space.
  name:
    type: string
    required: true
    description: Relationship name; must not duplicate an existing relationship name.
  relation_details:
    type: array
    required: true
    description: List of relationship details. Each must have project_key and work_item_type_key.
    items:
      type: object
      properties:
        project_key: { type: string }
        work_item_type_key: { type: string }

outputs:
  data:
    type: object
    description: relation_id (UUID of the newly added work item relationship).

constraints:
  - Permission: Permission Management – Work Item Instances
  - Operating user must have space administrator privileges (else 10001)

error_mapping:
  50006: Relationship name cannot be repeated (name unavailable or duplicate)
  10001: Invalid request (operating user is not admin of the space; contact admin for permissions)
```

### Usage notes

- **relation_details**: Each element specifies a target: **project_key** (target space) and **work_item_type_key** (target work item type). Same space or cross-space links are defined by these entries.
- **name**: Must be unique among work item relationships in the space; duplicate or invalid name returns 50006.
- **data.relation_id**: Use this UUID when creating custom fields of type work_item_related_select / work_item_related_multi_select (**work_item_relation_uuid**) or when updating/deleting this relationship.
- Space administrator permission is required (10001 if user is not admin of the space).

---

## Update Work Item Relationships

Update the configuration of the specified work item relationship. This interface performs **overwrite updates** (full replace of name and relation_details). Permission: Permission Management – Work Item Instance. For details, see Permission Management.

### Points to note

- This interface is for **overwrite updates**: the provided **name** and **relation_details** replace the existing configuration for the given **relation_id**.

### When to Use

- When changing the relationship name or the list of relation_details (target project_key and work_item_type_key) for an existing relationship
- When syncing relationship configuration from external config; use **relation_id** from Get the List of Work Item Relationships

### API Spec: update_work_item_relationships

```yaml
name: update_work_item_relationships
type: api
description: >
  Update the specified work item relationship (overwrite). Requires relation_id
  from Get the List of Work Item Relationships, plus project_key, work_item_type_key,
  name, and relation_details.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/work_item/relation/update
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params: {}

inputs:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.
  name:
    type: string
    required: true
    description: Relationship name; must not duplicate another relationship name.
  relation_id:
    type: string
    required: true
    description: Work item relationship ID from Get the List of Work Item Relationships.
  relation_details:
    type: array
    required: true
    description: List of relationship details. Each must have project_key and work_item_type_key.
    items:
      type: object
      properties:
        project_key: { type: string }
        work_item_type_key: { type: string }
        project_name: { type: string }
        work_item_type_name: { type: string }

outputs:
  data:
    type: object
    description: Empty on success (no data in response).

constraints:
  - Permission: Permission Management – Work Item Instance
  - Operating user must have space administrator privileges (else 10001)

error_mapping:
  50006: Relationship name cannot be repeated (name unavailable or duplicate)
  10001: Invalid request (operating user is not admin; contact admin for permissions)
```

### Usage notes

- **relation_id**: From **Get the List of Work Item Relationships** (data[].id). Identifies which relationship to update.
- **Overwrite**: The entire relationship config is replaced; send the full **name** and full **relation_details** list you want after the update.
- **relation_details**: Same structure as Create; each entry has **project_key** and **work_item_type_key** (and optionally project_name, work_item_type_name). Duplicate or invalid **name** returns 50006.
- Space administrator permission is required (10001 if not admin).

---

## Delete Work Item Relationships

Delete the work item association relationship identified by **relation_id** under the specified space. Permission: Permission Management – Work Item Instance. For details, see Permission Management.

### When to Use

- When removing a work item relationship (e.g. story → sprint) from the space
- When **relation_id** comes from Get the List of Work Item Relationships
- When cleaning up or reconfiguring relationships (delete then recreate if needed)

### API Spec: delete_work_item_relationships

```yaml
name: delete_work_item_relationships
type: api
description: >
  Delete the work item association relationship under the space. Requires
  project_key and relation_id from Get the List of Work Item Relationships.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: DELETE
  url: https://{domain}/open_api/work_item/relation/delete
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params: {}

inputs:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  relation_id:
    type: string
    required: true
    description: Work item relationship ID from Get the List of Work Item Relationships.

outputs:
  data:
    type: string
    description: Null or empty on success.

constraints:
  - Permission: Permission Management – Work Item Instance

error_mapping:
  50006: RPC call error (relationship already deleted or does not exist; confirm relation_id)
```

### Usage notes

- **relation_id**: Use the **id** from **Get the List of Work Item Relationships** (data[].id). If the relationship was already deleted or does not exist, 50006 is returned (err.msg may indicate “关系已经被删除” / relationship already deleted).
- Success response has empty or null **data**; check **err_code** 0.

---

## Create Workflow Role

Add a role under the specified work item type. Returns the new **Role ID**. Permission: Permission Management – Configuration.

### When to Use

- When creating a workflow role (e.g. Task, PM, DA) for a work item type
- When configuring **member_assign_mode** (manual / assign to specified person / assign to creator), **is_owner**, **auto_enter_group**, or **members** (user_key list)
- When you need the returned role ID for Get/Update/Delete Workflow Role or for node/field role bindings

### API Spec: create_workflow_role

```yaml
name: create_workflow_role
type: api
description: >
  Add a role under the specified work item type. Returns role ID. role object
  includes name, is_owner, auto_enter_group, member_assign_mode, members, is_member_multi, role_alias, lock_scope.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/{project_key}/flow_roles/{work_item_type_key}/create_role
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

inputs:
  role:
    type: object
    required: true
    description: Role configuration.
    properties:
      id:
        type: string
        description: Role ID. Optional; if not provided, auto-generated and returned on create.
      name:
        type: string
        description: Role name.
      is_owner:
        type: boolean
        description: Whether this role is the manager for the task.
      auto_enter_group:
        type: boolean
        description: Whether members are automatically added to the group.
      member_assign_mode:
        type: integer
        description: >
          1: Add manually; 2: Assign to a specified person by default; 3: Assign to the creator by default.
          When 2, members (user_key array) is used.
      members:
        type: array
        items: string
        description: Assigned members (user_key). Used when member_assign_mode is 2.
      is_member_multi:
        type: boolean
        description: Restrict to single-person configuration (false) or allow multiple.
      role_alias:
        type: string
        description: Role identifier/alias.
      lock_scope:
        type: array
        description: Lock scope (structure per product).

outputs:
  data:
    type: string
    description: Role ID (e.g. 5727769).

constraints:
  - Permission: Permission Management – Configuration

error_mapping:
  20006: Invalid param (role id already exists; change the id)
```

### Usage notes

- **role.id**: Optional; omit to let the server generate and return the role ID. If provided and already exists, 20006 is returned.
- **member_assign_mode**: **1** = manual add; **2** = assign to specified person (set **members** with user_key list); **3** = assign to creator by default.
- **members**: Only meaningful when **member_assign_mode** is **2**; values are **user_key** (from Meegle, e.g. double-click avatar in Developer Platform).
- **data**: Use the returned role ID when calling Get Detailed Role Settings, Update Workflow Role Settings, or Delete Workflow Role Configuration, or when binding roles to nodes/fields.

---

## Get Detailed Role Settings

Obtain the configuration of all roles and personnel under the specified work item type. Response follows the RelationDetail structure (id, name, is_owner, role_appear_mode, deletable, auto_enter_group, member_assign_mode, members, is_member_multi). Permission: Permission Management – Process Roles.

### When to Use

- When listing all workflow roles (e.g. PM, DA) for a work item type and their settings
- When you need role **id**, **member_assign_mode**, **members** (user_key list), or **deletable** for Update/Delete Workflow Role or for UI display
- When building role management UIs or syncing role configuration

### API Spec: get_detailed_role_settings

```yaml
name: get_detailed_role_settings
type: api
description: >
  Obtain all roles and personnel configuration under the specified work item type.
  Returns list per RelationDetail: id, name, is_owner, role_appear_mode, deletable,
  auto_enter_group, member_assign_mode, members, is_member_multi.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/flow_roles/{work_item_type_key}
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

outputs:
  data:
    type: array
    description: >
      List of RelationDetail. Each has id, name, is_owner, role_appear_mode,
      deletable, auto_enter_group, member_assign_mode, members (user_key array),
      is_member_multi.

constraints:
  - Permission: Permission Management – Process Roles

error_mapping:
  1000052062: Project key is wrong (project_key incorrect)
```

### Usage notes

- **data** items: Use **id** when updating or deleting a role (Update Workflow Role Settings, Delete Workflow Role Configuration). **deletable** indicates whether the role can be deleted.
- **member_assign_mode** and **members**: Same semantics as Create Workflow Role (1 manual, 2 specified members, 3 creator). **role_appear_mode**: Display/visibility mode per product.

---

## Update Workflow Role Settings

Update a role under the specified work item type. Identify the role by **role_id** or **role_alias** (one required; **role_id** takes precedence if both are sent). Permission: Permission Management – Process Roles.

### When to Use

- When changing role name, is_owner, auto_enter_group, member_assign_mode, members, is_member_multi, role_alias, or lock_scope
- When **role_id** or **role_alias** comes from Get Detailed Role Settings
- When member_assign_mode is 2, **members** must be provided (non-empty)

### API Spec: update_workflow_role_settings

```yaml
name: update_workflow_role_settings
type: api
description: >
  Update a role under the specified work item type. One of role_id or role_alias
  required (role_id preferred). role object contains updated config.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/{project_key}/flow_roles/{work_item_type_key}/update_role
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

inputs:
  role_id:
    type: string
    required: false
    description: Process role ID from Get Detailed Role Settings. One of role_id or role_alias required; role_id preferred if both sent.
  role_alias:
    type: string
    required: false
    description: Role docking identifier from Get Detailed Role Settings. One of role_id or role_alias required.
  role:
    type: object
    required: true
    description: Updated role configuration.
    properties:
      name: { type: string }
      is_owner: { type: boolean }
      auto_enter_group: { type: boolean }
      member_assign_mode: { type: integer }
      members: { type: array, items: string }
      is_member_multi: { type: boolean }
      role_alias: { type: string }
      lock_scope: { type: array }

outputs:
  data:
    type: object
    description: Empty on success.

constraints:
  - Permission: Permission Management – Process Roles
  - One of role_id or role_alias must be provided
  - When member_assign_mode is 2, members must be non-empty

error_mapping:
  20006: Invalid param (members should not be empty when member_assign_mode is 2)
  20006: Invalid param (role_id does not exist; obtain via Get Detailed Role Settings)
```

### Usage notes

- **role_id** or **role_alias**: Provide one to identify the role to update; get from **Get Detailed Role Settings** (data[].id or role_alias). If both are sent, **role_id** is used.
- **role**: Same shape as Create Workflow Role (name, is_owner, auto_enter_group, member_assign_mode, members, is_member_multi, role_alias, lock_scope). When **member_assign_mode** is **2**, **members** must be a non-empty user_key array (20006 otherwise).

---

## Delete Workflow Role Configuration

Delete a role under the specified work item type. Identify the role by **role_id** or **role_alias** (one required; **role_id** takes precedence if both are sent). Permission: Permission Management – Configuration.

### When to Use

- When removing a workflow role (e.g. PM, DA) from a work item type
- When **role_id** or **role_alias** comes from Get Detailed Role Settings
- Role must not be in use in process template nodes (else 20093)

### API Spec: delete_workflow_role_configuration

```yaml
name: delete_workflow_role_configuration
type: api
description: >
  Delete a role under the specified work item type. One of role_id or role_alias
  required (role_id preferred). Role cannot be referenced in template nodes.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/{project_key}/flow_roles/{work_item_type_key}/delete_role
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

inputs:
  role_id:
    type: string
    required: false
    description: Process role ID from Get Detailed Role Settings. One of role_id or role_alias required; role_id preferred if both sent.
  role_alias:
    type: string
    required: false
    description: Role docking identifier from Get Detailed Role Settings. One of role_id or role_alias required.

outputs:
  data:
    type: object
    description: Empty on success.

constraints:
  - Permission: Permission Management – Configuration
  - One of role_id or role_alias must be provided
  - Role must not be referenced in process template nodes (else 20093)

error_mapping:
  20093: Role in use (cannot delete; role is referenced in template nodes)
  20006: Invalid param (role_id does not exist or has been deleted; obtain via Get Detailed Role Settings)
```

### Usage notes

- **role_id** or **role_alias**: Provide one to identify the role to delete; get from **Get Detailed Role Settings**. If both are sent, **role_id** is used.
- **20093**: The role is referenced in workflow template nodes and cannot be deleted until those references are removed.
- **20006**: The given role_id does not exist or was already deleted; confirm via Get Detailed Role Settings.

---

## Get Workflow Templates

Obtain the list of all process templates under the specified work item type. Response follows the TemplateConf structure (version, unique_key, is_disabled, template_id, template_key, template_name). Permission: Permission Management – Configuration.

### When to Use

- When listing workflow/process templates for a work item type (e.g. story, issue)
- When you need **template_id**, **unique_key**, or **template_name** for Get Detailed Settings of Workflow Templates, Update Workflow Template, or Delete Workflow Templates
- When building template selection UIs or checking **is_disabled** / **version** per template

### API Spec: get_workflow_templates

```yaml
name: get_workflow_templates
type: api
description: >
  Obtain the list of all process templates under the specified work item type.
  Returns list per TemplateConf: version, unique_key, is_disabled, template_id, template_key, template_name.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/template_list/{work_item_type_key}
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.

outputs:
  data:
    type: array
    description: >
      List of TemplateConf. Each has version, unique_key, is_disabled, template_id,
      template_key, template_name.

constraints:
  - Permission: Permission Management – Configuration
```

### Usage notes

- **data** items: Use **template_id** or **unique_key** when calling Get Detailed Settings of Workflow Templates, Update Workflow Template, or Delete Workflow Templates. **is_disabled** indicates whether the template is disabled (value meaning per product).
- For common error codes and server-side call analysis, refer to **Open API Error Codes** in the product docs.

---

## Get Detailed Settings of Workflow Templates

Obtain the detailed configuration of the specified process template, including node information and node transfer configuration. Node event configuration is not supported. Response follows the TemplateDetail structure (template_id, template_name, version, work_item_type_key, workflow_confs, connections). Permission: Permission Management – Process Type.

### When to Use

- When editing or displaying a workflow template’s nodes, fields, sub_tasks, sub_work_items, and transitions (connections)
- When **template_id** comes from Get Workflow Templates, Get metadata for creating work items, or Get field information (template field options)
- When building template editors or syncing workflow config; omit template_id to use the first process template of the work item type (per product behavior)

### API Spec: get_detailed_settings_of_workflow_templates

```yaml
name: get_detailed_settings_of_workflow_templates
type: api
description: >
  Obtain detailed configuration of the specified process template: node info and
  node transfer config. TemplateDetail includes workflow_confs (nodes) and connections.
  Node event configuration not supported.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: GET
  url: https://{domain}/open_api/{project_key}/template_detail/{template_id}
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  template_id:
    type: string
    required: true
    description: >
      Process template ID. Obtain from Get Workflow Templates, or from template
      field options in Get metadata for creating work items / Get field information.
      When not passed, first process template of the work item type may be used (product-dependent).

outputs:
  data:
    type: object
    description: >
      TemplateDetail: template_id, template_name, version, is_disabled,
      work_item_type_key, workflow_confs (nodes with name, state_key, owner_usage_mode,
      fields, sub_tasks, sub_work_items, done_operation_role, connections, etc.),
      connections (source_state_key, target_state_key).

constraints:
  - Permission: Permission Management – Process Type
  - Node event configuration not supported

error_mapping:
  50006: Template not found (check template_id)
  30001: Data not found (template_id not found in project_key space)
```

### Usage notes

- **template_id**: From **Get Workflow Templates** (data[].template_id), or from the template field options in **Get metadata for creating work items** / **Get field information**. Must belong to the space identified by **project_key** (30001 otherwise).
- **data.workflow_confs**: Array of node configs; each has **state_key**, **name**, **fields**, **sub_tasks**, **sub_work_items**, **done_operation_role**, **connections**, and related flags. **data.connections**: Transitions between state_key values.
- Node event configuration is not supported in this interface.

---

## Create Workflow Templates

Create a new process template under the specified work item type. Optionally copy from an existing template via **copy_template_id**. Returns the new **template ID**. Permission: Permission Management – Configuration.

### When to Use

- When creating a new process template for a work item type (e.g. story, issue)
- When copying an existing template: set **copy_template_id** from Get Workflow Templates (list of process templates under the work item type)
- When you need the returned template ID for Get Detailed Settings, Update, or Delete Workflow Template

### API Spec: create_workflow_templates

```yaml
name: create_workflow_templates
type: api
description: >
  Create a new process template under the specified work item type. Optional
  copy_template_id to reuse an existing template. Returns new template ID (int64).

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: POST
  url: https://{domain}/open_api/template/v2/create_template
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params: {}

inputs:
  project_key:
    type: string
    required: true
    description: Space ID (project_key). Double-click space name in Meegle project space.
  work_item_type_key:
    type: string
    required: true
    description: Work item type. Obtain via Get work item types in space.
  template_name:
    type: string
    required: true
    description: Process template name; must not duplicate an existing template name.
  copy_template_id:
    type: integer
    required: false
    description: >
      ID of template to copy from. Obtain via Get Workflow Templates (list of process
      templates under the work item type). Omit to create an empty new template.

outputs:
  data:
    type: integer
    description: Template ID of the newly created process template (int64).

constraints:
  - Permission: Permission Management – Configuration
  - Current user must have space configuration permission (else 10005)

error_mapping:
  10005: No project admin permission (no permission to configure the space)
  50006: Template name duplicate (template name already exists; change the name)
```

### Usage notes

- **template_name**: Must be unique among process templates for the work item type in the space; duplicate returns 50006.
- **copy_template_id**: From **Get Workflow Templates** (data[].template_id). When provided, the new template is created by copying this template; when omitted, a new empty template is created.
- **data**: Use the returned template ID when calling Get Detailed Settings of Workflow Templates, Update Workflow Template, or Delete Workflow Templates.
- Space configuration (admin) permission is required (10005 if not allowed).

---

## Update Workflow Template

Update the configuration of the specified process template. Supports node flow (**workflow_confs**, WorkflowConf) and state flow (**state_flow_confs**, StateFlowConf). Permission: Permission Management – Settings.

### When to Use

- When changing node configuration (add/delete/modify nodes via **action** in workflow_confs or state_flow_confs)
- When **template_id** comes from Get Workflow Templates, Get metadata for creating work items, or Get field information
- When updating owner_usage_mode, owner_roles, owners, need_schedule, pass_mode, done_operation_role, task_confs, etc.

### API Spec: update_workflow_template

```yaml
name: update_workflow_template
type: api
description: >
  Update the specified process template. workflow_confs for node flow (WorkflowConf),
  state_flow_confs for state flow (StateFlowConf). Node action: 1 Add, 2 Delete (state_key only), 3 Modify.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: PUT
  url: https://{domain}/open_api/template/v2/update_template
  headers:
    Content-Type: application/json
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params: {}

inputs:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  template_id:
    type: integer
    required: true
    description: >
      Process template ID. From Get Workflow Templates, or template field options in
      Get metadata for creating work items / Get field information.
  workflow_confs:
    type: array
    required: false
    description: Node flow configuration per WorkflowConf. Each can have action (1 add, 2 delete, 3 modify), state_key, name, tags, pre_node_state_key, owner_usage_mode, owner_roles, owners, need_schedule, different_schedule, deletable, deletable_operation_role, pass_mode, done_operation_role, done_schedule, done_allocate_owner, task_confs, etc. For action 2 (delete), only state_key required.
  state_flow_confs:
    type: array
    required: false
    description: State flow configuration per StateFlowConf structure.

outputs:
  data:
    type: object
    description: Empty on success (no data in response).

constraints:
  - Permission: Permission Management – Settings
  - Current user must have space configuration permission (else 10005)

error_mapping:
  10005: No project admin permission (operator cannot configure the space)
  1000052062: Project key is wrong (project_key incorrect)
  20006: Invalid param (state_key illegal for node flow work item)
  50006: Template not found (template_id incorrect)
```

### Usage notes

- **template_id**: From **Get Workflow Templates**, or from template field options in **Get metadata for creating work items** / **Get field information**. Must be correct for the space (50006 if not found).
- **workflow_confs** (node flow): **action** 1 = add node, 2 = delete node (only **state_key** needed), 3 = modify node. Structure follows WorkflowConf (state_key, name, tags, owner_usage_mode, owner_roles, owners, need_schedule, different_schedule, pass_mode, done_operation_role, task_confs, etc.).
- **state_flow_confs**: State flow config per StateFlowConf; use for state-flow work item types.
- Space configuration (admin) permission is required (10005 if not allowed).

---

## Delete Workflow Templates

Delete the specified process template. Permission: Permission Management – Settings.

### When to Use

- When removing a process template for a work item type
- When **template_id** comes from Get Workflow Templates, Get metadata for creating work items, or Get field information (template field options)

### API Spec: delete_workflow_templates

```yaml
name: delete_workflow_templates
type: api
description: >
  Delete the specified process template. Path parameters project_key and template_id.

auth:
  type: plugin_access_token
  header: X-Plugin-Token
  user_header: X-User-Key

http:
  method: DELETE
  url: https://{domain}/open_api/template/v2/delete_template/{project_key}/{template_id}
  headers:
    X-Plugin-Token: "{{resolved_token}}"
    X-User-Key: "{{user_key}}"

path_params:
  project_key:
    type: string
    required: true
    description: >
      Space ID (project_key) or space domain name (simple_name).
      project_key: Double-click space name in Meegle. simple_name: from space URL (e.g. doc).
  template_id:
    type: string
    required: true
    description: >
      Process template ID. From Get Workflow Templates, or template field options in
      Get metadata for creating work items / Get field information.

outputs:
  data:
    type: object
    description: Empty on success (no data in response).

constraints:
  - Permission: Permission Management – Settings

error_mapping:
  9999: Invalid param (key template_id value invalid; template_id incorrect)
```

### Usage notes

- **template_id**: From **Get Workflow Templates** (data[].template_id), or from template field options in **Get metadata for creating work items** / **Get field information**. Invalid or non-existent template_id returns 9999.
- Success response has no **data**; check **err_code** 0.
