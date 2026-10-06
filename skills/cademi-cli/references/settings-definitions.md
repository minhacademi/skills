---
name: cademi-cli-settings-definitions
description: "Shared definitions: custom fields, tags, custom code and declarative configuration — `cademi settings` (18 commands)"
metadata:
  cademi-cli: "0.2.5"
  cademi-api: "3.12.0"
---

# Settings Definitions Commands

> cademi 0.2.5, API 3.12.0. The live catalog is always `cademi commands <prefix> --json`.

Shared definitions: custom fields, tags, custom code and declarative configuration. The rest of `cademi settings` is in the sibling files below.

**Related:** `cademi config`, `cademi users tags`, `cademi users custom-fields`

**See also in this domain:** `references/settings-access.md`, `references/settings-platform.md`, `references/settings-emails.md`, `references/settings-legal-terms.md`, `references/settings-menus.md`, `references/settings-support.md`

Live catalog for this file: `cademi commands settings --json` (offline, no credential needed).

## Shared flag sets

Each command lists the sets it accepts. The flags of a set are:

**output**
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document)
- `--jq <string>` — Filter data (or --raw envelope); strings print unquoted, other values as JSON; overrides --output
- `--json` — Print JSON (same as --output json)
- `-o, --output <string>` — Output format: json, table or yaml (default: table for lists on a terminal, JSON otherwise)
- `--raw` — Print the full response envelope instead of data

**output-basic**
- `--jq <string>` — Filter data (or --raw envelope); strings print unquoted, other values as JSON; overrides --output
- `--json` — Print JSON (same as --output json)
- `-o, --output <string>` — Output format: json, table or yaml (default: table for lists on a terminal, JSON otherwise)

**body**
- `-d, --data <string>` — JSON body: a literal, @file or @- for stdin
- `-f, --field <stringArray>` — Add a string field: key=value (nested: a.b=v, arrays: tags[]=v)
- `-F, --typed-field <stringArray>` — Add a typed field: key=true\|false\|null\|number\|JSON, or key=@file to read text; applied after all -f fields

**idempotency**
- `--idempotency-key <string>` — Reuse the key returned with a failed request to repeat it (default: automatic UUIDv7)

**async**
- `--wait` — When the API answers 202, wait for the operation to finish
- `--wait-timeout <duration>` — Maximum time for --wait, e.g. 30s or 2m (default: no limit; does not cancel the operation)

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi settings configuration applications` — Apply a configuration plan

#### `cademi settings configuration applications create`

Apply a configuration plan · `POST /api/v3/configuration/applications` · permission `configuration.apply`

Applies a configuration plan previously returned by the plan operation. Submit the plan's `id` as `plan_id`, together with its `signature` and `steps`.

Before accepting the request, the API verifies that the plan is still valid, that its signature matches the submitted steps, and that none of the affected resources has changed since the plan was calculated. A plan can only be applied with the same credentials used to create it, and only once.

The plan is applied asynchronously. The response returns an operation, and the `Location` header points to it so you can track progress.

Steps are not applied as a single transaction: if a step fails, steps that were already applied remain in effect.

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `plan_id` | string | yes |  |
| `signature` | string | yes | The `signature` value returned with the plan. |
| `steps` | array of object | yes | The plan steps exactly as returned with the plan. Write-only values appear as `"***"` in the plan; replace them with the actual values before submitting. This replacement does not invalidate the signature. Submitting `"***"` as a value is… |

Full schema: `cademi commands settings configuration applications create --schema --json`

Legacy path: `cademi configuration applications create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings configuration applications create -f plan_id=<plan_id> -f signature=<signature> --data '{"steps":[...]}' --wait --json

# full body from a file
cademi settings configuration applications create --data @body.json --wait --json
```

### `cademi settings configuration plans` — Create a configuration plan

#### `cademi settings configuration plans create`

Create a configuration plan · `POST /api/v3/configuration/plans` · permission `configuration.plan`

Calculates the steps required to bring the account to the state described in a configuration manifest, and returns them as a signed plan.

This operation is synchronous and does not modify any resource. The plan expires 24 hours after it is created, as indicated by `expires_at`. To apply it, submit the plan to the apply operation before it expires.

If the manifest is invalid, the API returns `validation_failed` with the same error codes reported by the validation operation, and no plan is created.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string |  |  |
| `order` | object |  | Ordered sets keyed by set name (for example, `showcases` or `menu_items`). Each value must be the complete list for its scope. |
| `resources` | array of object | yes |  |

Full schema: `cademi commands settings configuration plans create --schema --json`

Legacy path: `cademi configuration plans create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings configuration plans create --data '{"resources":[...]}' --json

# full body from a file
cademi settings configuration plans create --data @body.json --json
```

### `cademi settings configuration validations` — Validate a configuration manifest

#### `cademi settings configuration validations create`

Validate a configuration manifest · `POST /api/v3/configuration/validations` · permission `configuration.validate`

Validates a configuration manifest without modifying any resource.

Content problems, such as invalid fields, unresolved references, or incomplete ordered sets (`order_set_mismatch`), are reported in the validation report, and the response status is still `200`. Check the `valid` field to determine the result.

The request is rejected with `422` only when the manifest cannot be evaluated: `too_many_items` when it exceeds the maximum number of resources, and `unsupported_resource` when it contains an unsupported resource kind.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `$schema` | string |  |  |
| `order` | object |  | Ordered sets keyed by set name (for example, `showcases` or `menu_items`). Each value must be the complete list for its scope. |
| `resources` | array of object | yes |  |

Full schema: `cademi commands settings configuration validations create --schema --json`

Legacy path: `cademi configuration validations create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings configuration validations create --data '{"resources":[...]}' --json

# full body from a file
cademi settings configuration validations create --data @body.json --json
```

### `cademi settings custom-code` — Manage custom scripts and styles

Manage custom code applied to the platform's presentation.

Related settings:
- `cademi settings branding` — Controls the student-area visual identity and certificate logos. (`cademi settings branding get|update`)

#### `cademi settings custom-code create`

Create a custom code block · `POST /api/v3/settings/custom-code` · permission `custom_code.write`, `custom_code.manage_pwa (if transition:type=pushalertco)`

Creates an active custom code block. Creation enables it immediately; no separate activation request is needed. The create operation does not accept status.

Blocks created through this API have no login-only restriction. Their integration renderer is used by student-area and sign-in page layouts; OAuth consent sign-in and two-factor pages suppress instance scripts. The integration type determines the rendered code.

Creating a block with `type` set to `pushalertco` also requires the `custom_code.manage_pwa` permission.

Code content is stored exactly as submitted and is not sanitized.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `content` | object |  | Code content. Keys that are omitted are stored empty. `sw_js` and `manifest_json` are used only by `pushalertco` blocks. |
| `content.body` | string, nullable |  | Code injected into the page `<body>`. |
| `content.header` | string, nullable |  | Code injected into the page `<head>`. |
| `content.manifest_json` | string, nullable |  | Web app manifest. |
| `content.sw_js` | string, nullable |  | Service worker script. |
| `name` | string | yes | Identifies the block in this list. Students never see this name. |
| `position` | string | yes | Required input accepted as `head` or `body`, but it does not select the injection location. Set `content.header` and/or `content.body` to populate those locations. The returned position is derived from the saved content: `head` when… |
| `type` | string | yes | Selects the integration implementation. Accepted values: `active-campaign`, `custom`, `facebook-pixel`, `analytics`, `tagmanager`, `pushalertco`, `whats`. Creating a `pushalertco` block also requires `custom_code.manage_pwa`. The API… |

Full schema: `cademi commands settings custom-code create --schema --json`

Legacy path: `cademi custom-code create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings custom-code create -f name=<name> -f position=head -f type=<type> --json

# full body from a file
cademi settings custom-code create --data @body.json --json
```

#### `cademi settings custom-code delete <custom_code_id>`

Delete a custom code block · `DELETE /api/v3/settings/custom-code/{custom_code_id}` · permission `custom_code.write`

Moves the custom code block to the trash instead of permanently deleting it.

A block in the trash can be restored with the update operation by sending `deleted: false`.

**Arguments:**
- `custom_code_id` — Public ID of the custom code, prefixed with `cc_`. Example: `cc_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi custom-code delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings custom-code delete cc_42 --yes
```

#### `cademi settings custom-code get <custom_code_id>`

Retrieve a custom code block · `GET /api/v3/settings/custom-code/{custom_code_id}` · permission `custom_code.read`

Retrieves a custom code block by its public ID, including blocks in the trash.

The block metadata is always returned. The `content` object (`header`, `body`, `sw_js`, and `manifest_json`) is included only when the credentials also have the `custom_code.read_content` permission, because code content often contains third-party tokens.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the block to avoid overwriting a newer version.

**Arguments:**
- `custom_code_id` — Public ID of the custom code, prefixed with `cc_`. Example: `cc_42`

**Flag sets:** output

Legacy path: `cademi custom-code get`

**Examples:**

```bash
# get
cademi settings custom-code get cc_42 --json
```

#### `cademi settings custom-code list`

List custom code blocks · `GET /api/v3/settings/custom-code` · permission `custom_code.read`

Returns the metadata of the account's custom code blocks, using cursor-based pagination.

By default, only blocks that are not in the trash are returned. Set `deleted=true` to list only blocks in the trash.

The `content` object is never included in this collection. To read a block's code, use the retrieve operation with the `custom_code.read_content` permission.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

Legacy path: `cademi custom-code list`

**Examples:**

```bash
# list: one page, machine-readable
cademi settings custom-code list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi settings custom-code list --limit 200 --raw --json

# every page, projected
cademi settings custom-code list --all --jq '[.[] | {id}]'
```

#### `cademi settings custom-code update <custom_code_id>`

Update a custom code block · `PATCH /api/v3/settings/custom-code/{custom_code_id}` · permission `custom_code.write`, `custom_code.activate (if field:status)`

Updates an existing custom code block. Only the fields present in the request are changed.

Changing `status` also requires the `custom_code.activate` permission. Changing the `name` or `content` of a `pushalertco` block also requires the `custom_code.manage_pwa` permission.

Send `deleted: false` to restore a block from the trash.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting changes made since the block was retrieved.

**Arguments:**
- `custom_code_id` — Public ID of the custom code, prefixed with `cc_`. Example: `cc_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `content` | object |  | Code content. Only the keys included are changed. |
| `content.body` | string, nullable |  | Code injected into the page `<body>`. |
| `content.header` | string, nullable |  | Code injected into the page `<head>`. |
| `content.manifest_json` | string, nullable |  | Web app manifest. |
| `content.sw_js` | string, nullable |  | Service worker script. |
| `deleted` | boolean |  | Send `false` to restore a block from the trash. One of: `false` |
| `name` | string |  | Identifies the block in this list. Students never see this name. |
| `status` | string |  | `active` enables the block; `inactive` keeps it as a draft. Changing this field also requires `custom_code.activate`. While in draft, the code is not injected into the pages. One of: `active`, `inactive` |

Full schema: `cademi commands settings custom-code update --schema --json`

**Additional permission checks:**
- `custom_code.manage_pwa` when `stored:type=pushalertco AND any-field:name,content`

Legacy path: `cademi custom-code update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings custom-code update cc_42 -f content.body=<content.body> --if-match '"<etag>"' --json

# full body from a file
cademi settings custom-code update cc_42 --data @body.json --json
```

### `cademi settings custom-fields` — Define custom fields for student profiles

Manage field definitions and types. Read or update a student's values
through users custom-fields.

Related: `cademi users custom-fields`

#### `cademi settings custom-fields create`

Create a custom field · `POST /api/v3/settings/user-profile/custom-fields` · permission `custom_fields.create`

Creates a custom field definition for user profiles in the current account.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `label` | string | yes | Field name shown when viewing or editing a student profile. |
| `position` | integer |  | Nonnegative ordering value for the custom field definition list. Lower values appear first, with the field ID breaking ties. Omit to place the new definition after the existing definitions. |
| `required` | boolean |  | Stores whether the field is marked as required. Current user creation, custom-field value updates and student profile forms do not enforce this flag: omitted or empty values remain allowed. Omit to create the definition with this flag set… |
| `type` | string | yes | Profile input type. When updating values through `users.custom_fields.update`, `number` requires a numeric value and `date` requires `YYYY-MM-DD`; `text` accepts text. One of: `text`, `number`, `date` |

Full schema: `cademi commands settings custom-fields create --schema --json`

Legacy path: `cademi custom-fields create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings custom-fields create -f label=<label> -f type=text --json

# full body from a file
cademi settings custom-fields create --data @body.json --json
```

#### `cademi settings custom-fields delete <custom_field_id>`

Delete a custom field · `DELETE /api/v3/settings/user-profile/custom-fields/{custom_field_id}` · permission `custom_fields.delete`

Deletes a custom field definition.

Only the definition is removed. Values that users have already filled in for this field are retained but are no longer returned with the user's custom field values.

**Arguments:**
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi custom-fields delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings custom-fields delete cfd_7 --yes
```

#### `cademi settings custom-fields get <custom_field_id>`

Retrieve a custom field · `GET /api/v3/settings/user-profile/custom-fields/{custom_field_id}` · permission `custom_fields.read`

Retrieves a custom field definition by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the custom field to avoid overwriting a newer version.

**Arguments:**
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output

Legacy path: `cademi custom-fields get`

**Examples:**

```bash
# get
cademi settings custom-fields get cfd_7 --json
```

#### `cademi settings custom-fields list`

List custom fields · `GET /api/v3/settings/user-profile/custom-fields` · permission `custom_fields.read`

Lists the custom field definitions available for user profiles in the current account.

Returns all definitions in a single response, ordered by `position`.

**Flag sets:** output

Legacy path: `cademi custom-fields list`

**Examples:**

```bash
# get
cademi settings custom-fields list --json
```

#### `cademi settings custom-fields update <custom_field_id>`

Update a custom field · `PATCH /api/v3/settings/user-profile/custom-fields/{custom_field_id}` · permission `custom_fields.update`

Updates a custom field definition. Fields omitted from the request keep their current values.

The `type` of a field cannot be changed once users have filled in values for it. Such requests return the `state_conflict` error code.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header.

**Arguments:**
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `label` | string |  | Field name shown when viewing or editing a student profile. |
| `position` | integer |  | Nonnegative ordering value for the custom field definition list. Lower values appear first, with the field ID breaking ties. Omit to preserve the current position. |
| `required` | boolean |  | Stores whether the field is marked as required. Current user creation, custom-field value updates and student profile forms do not enforce this flag: omitted or empty values remain allowed. Omit to preserve the current flag. |
| `type` | string |  | Profile input type. When updating values through `users.custom_fields.update`, `number` requires a numeric value and `date` requires `YYYY-MM-DD`; `text` accepts text. Changing the type is rejected with `state_conflict` if any user value… |

Full schema: `cademi commands settings custom-fields update --schema --json`

Legacy path: `cademi custom-fields update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings custom-fields update cfd_7 -f label=<label> --if-match '"<etag>"' --json

# full body from a file
cademi settings custom-fields update cfd_7 --data @body.json --json
```

### `cademi settings tags` — Define tags used to classify students

Create and edit tag definitions here. Assign tags to a student through
users tags; deliveries may also apply tags when granting access.

Related: `cademi users tags`, `cademi sales deliveries tags`

#### `cademi settings tags create`

Create a tag · `POST /api/v3/tags` · permission `tags.create`

Creates a tag that can be assigned to users.

Leading and trailing whitespace is removed from the name. Tag names must be unique within the account; if the name is already in use, the API returns `409 Conflict` with the `already_exists` error code.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes |  |

Full schema: `cademi commands settings tags create --schema --json`

Legacy path: `cademi tags create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings tags create -f name=<name> --json

# full body from a file
cademi settings tags create --data @body.json --json
```

#### `cademi settings tags delete <tag_id>`

Delete a tag · `DELETE /api/v3/tags/{tag_id}` · permission `tags.delete`

Moves the tag to the trash and removes it from every user it is assigned to.

A tag used by an access rule cannot be deleted; the API returns `409 Conflict` with the `tag_in_use` error code.

Replicated tags are read-only and cannot be deleted. Attempts to delete them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi tags delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi settings tags delete tag_42 --yes
```

#### `cademi settings tags get <tag_id>`

Retrieve a tag · `GET /api/v3/tags/{tag_id}` · permission `tags.read`

Retrieves a tag by its public ID, including the number of users it is assigned to.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the tag to avoid overwriting a newer version.

**Arguments:**
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_42`

**Flag sets:** output

Legacy path: `cademi tags get`

**Examples:**

```bash
# get
cademi settings tags get tag_42 --json
```

#### `cademi settings tags list`

List tags · `GET /api/v3/tags` · permission `tags.read`

Lists the user tags of the account, sorted alphabetically by name by default. Each tag includes `users_count`, the number of users it is assigned to. Results are paginated with `limit` and `cursor`.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (name, -name); API default: name

Legacy path: `cademi tags list`

**Examples:**

```bash
# list: one page, machine-readable
cademi settings tags list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi settings tags list --limit 200 --raw --json

# every page, projected
cademi settings tags list --all --jq '[.[] | {id}]'
```

#### `cademi settings tags update <tag_id>`

Update a tag · `PATCH /api/v3/tags/{tag_id}` · permission `tags.update`

Renames a tag. The new name must be unique within the account.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header. If the header is omitted, the update is applied to the current revision.

Replicated tags are read-only. Attempts to update them return `403 Forbidden` with the `replica_readonly` error code.

**Arguments:**
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string |  |  |

Full schema: `cademi commands settings tags update --schema --json`

Legacy path: `cademi tags update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi settings tags update tag_42 -f name=<name> --if-match '"<etag>"' --json

# full body from a file
cademi settings tags update tag_42 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
