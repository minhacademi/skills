---
name: cademi-cli-users-profile
description: "Tags, custom field values, notes, attribution, avatar and term acceptances of a user — `cademi users` (17 commands)"
metadata:
  cademi-cli: "0.2.1"
  cademi-api: "3.10.0"
---

# Users Profile Commands

> cademi 0.2.1, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Tags, custom field values, notes, attribution, avatar and term acceptances of a user. The rest of `cademi users` is in the sibling files below.

**Related:** `cademi sales deliveries`

**Related settings:**
- `cademi settings registration` — Controls free sign-up channels, the delivery granted to new students, allowed email domains and document requirements. (`cademi settings registration get|update`)
- `cademi settings user-profile` — Controls which profile fields students can see or edit and how student names appear in support tools. (`cademi settings user-profile get|update`)
- `cademi settings tags` — Create and edit tag definitions here. Assign tags to a student through users tags; deliveries may also apply tags when granting access. (`cademi settings tags create|delete|get|list|update`)
- `cademi settings custom-fields` — Manage field definitions and types. Read or update a student's values through users custom-fields. (`cademi settings custom-fields create|delete|get|list|update`)

**See also in this domain:** `references/users-learning.md`, `references/users.md`, `references/users-imports.md`, `references/users-products.md`

**Group examples:**

```bash
cademi users list
cademi users enrollments list usr_42
cademi users products access get usr_42 prd_7
```

Live catalog for this file: `cademi commands users --json` (offline, no credential needed).

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

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi users attribution` — Manage users attribution

#### `cademi users attribution get <user_id>`

Retrieve user attribution · `GET /api/v3/users/{user_id}/attribution` · permission `users.read`

Retrieves the attribution data captured when the user signed up, such as UTM parameters, traffic source, company, and job title. Fields without a value are returned as `null`.

If extra sign-up fields are not enabled for the account, the API returns `feature_disabled`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users attribution get usr_42 --json
```

#### `cademi users attribution update <user_id>`

Update user attribution · `PUT /api/v3/users/{user_id}/attribution` · permission `users.update`

Saves the user's sign-up attribution data. Fields that are not supported attribution fields are ignored.

If extra sign-up fields are not enabled for the account, the request is rejected with `feature_disabled` instead of being ignored.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `adcampaign` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `adid` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `cargo` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `fbclid` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `gclid` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `groupid` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `nome_empresa` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `page_name` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `tamanho_empresa` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `utm_campaign` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `utm_content` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `utm_medium` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `utm_source` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |
| `utm_term` | string \| number \| boolean, nullable |  | Attribution value. Send a string; scalar values are converted to trimmed text. null or an empty string clears the value. |

Full schema: `cademi commands users attribution update --schema --json`

**Examples:**

```bash
# partial update
cademi users attribution update usr_42 -f adcampaign=<adcampaign> --json

# full body from a file
cademi users attribution update usr_42 --data @body.json --json
```

### `cademi users avatar` — Manage users avatar

#### `cademi users avatar delete <user_id>`

Delete a user avatar · `DELETE /api/v3/users/{user_id}/avatar` · permission `users.update`

Removes the user's avatar. The operation is idempotent: removing an avatar that is not set also succeeds.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, idempotency, confirm

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users avatar delete usr_42 --yes
```

#### `cademi users avatar update <user_id>`

Update a user avatar · `PUT /api/v3/users/{user_id}/avatar` · permission `users.update`

Sets the user's avatar to a file previously uploaded to the account, identified by `file_id`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `file_id` | string | yes |  |

Full schema: `cademi commands users avatar update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users avatar update usr_42 -f file_id=<file_id> --json

# full body from a file
cademi users avatar update usr_42 --data @body.json --json
```

### `cademi users custom-fields` — Manage custom field values for one user

These operations manage a user's values. Field definitions, labels and
types are managed through settings custom-fields.

Related settings:
- `cademi settings custom-fields` — Manage field definitions and types. Read or update a student's values through users custom-fields. (`cademi settings custom-fields create|delete|get|list|update`)

#### `cademi users custom-fields delete <user_id> <custom_field_id>`

Delete a custom field value · `DELETE /api/v3/users/{user_id}/custom-fields/{custom_field_id}` · permission `users.update`

Clears the user's value for a custom field. The custom field itself is not affected.

The operation is idempotent: clearing a value that is not set also succeeds.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `custom_field_id` — Public ID of the custom field, prefixed with `cfd_`. Example: `cfd_7`

**Flag sets:** output, idempotency, confirm

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users custom-fields delete usr_42 cfd_7 --yes
```

#### `cademi users custom-fields list <user_id>`

List user custom field values · `GET /api/v3/users/{user_id}/custom-fields` · permission `users.read`

Returns the user's custom field values, keyed by custom field public ID. Values of deleted custom fields are not included.

If custom fields are not enabled for the account, the API returns `feature_disabled`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users custom-fields list usr_42 --json
```

#### `cademi users custom-fields update <user_id>`

Update user custom field values · `PUT /api/v3/users/{user_id}/custom-fields` · permission `users.update`

Sets custom field values for the user. Send a `values` object keyed by custom field public ID. Custom fields not included in the request keep their current values.

Values are validated against the field type: `number` fields require a numeric value and `date` fields require the `YYYY-MM-DD` format. Invalid values return `validation_failed`, with the affected field identified in `details[].field`.

If custom fields are not enabled for the account, the API returns `feature_disabled`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `values` | object | yes |  |

Full schema: `cademi commands users custom-fields update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users custom-fields update usr_42 -F 'values={...}' --json

# full body from a file
cademi users custom-fields update usr_42 --data @body.json --json
```

### `cademi users notes` — Manage users notes

#### `cademi users notes create <user_id>`

Create a user note · `POST /api/v3/users/{user_id}/notes` · permission `users.update`

Adds an internal note about the user. The note is attributed to the credential that created it, and only that credential can later edit or delete it.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | string | yes |  |

Full schema: `cademi commands users notes create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users notes create usr_42 -f body=<body> --json

# full body from a file
cademi users notes create usr_42 --data @body.json --json
```

#### `cademi users notes delete <user_id> <note_id>`

Delete a user note · `DELETE /api/v3/users/{user_id}/notes/{note_id}` · permission `users.update`

Deletes an internal note. Only the author of the note can delete it; other callers receive `permission_denied`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `note_id` — Public ID of the note, prefixed with `note_`. Example: `note_9`

**Flag sets:** output, idempotency, confirm

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users notes delete usr_42 note_9 --yes
```

#### `cademi users notes get <user_id> <note_id>`

Retrieve a user note · `GET /api/v3/users/{user_id}/notes/{note_id}` · permission `users.read`

Retrieves an internal note about the user.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `note_id` — Public ID of the note, prefixed with `note_`. Example: `note_9`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users notes get usr_42 note_9 --json
```

#### `cademi users notes list <user_id>`

List user notes · `GET /api/v3/users/{user_id}/notes` · permission `users.read`

Returns the internal notes recorded about the user.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users notes list usr_42 --json
```

#### `cademi users notes update <user_id> <note_id>`

Update a user note · `PATCH /api/v3/users/{user_id}/notes/{note_id}` · permission `users.update`

Updates the body of an internal note. Only the author of the note, whether an administrator or the credential that created it, can edit it; other callers receive `permission_denied`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `note_id` — Public ID of the note, prefixed with `note_`. Example: `note_9`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | string | yes |  |

Full schema: `cademi commands users notes update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users notes update usr_42 note_9 -f body=<body> --json

# full body from a file
cademi users notes update usr_42 note_9 --data @body.json --json
```

### `cademi users tags` — Manage tag assignments for one user

Supply a user ID to inspect or change their assigned tags. Create and edit
tag definitions through settings tags.

Related settings:
- `cademi settings tags` — Create and edit tag definitions here. Assign tags to a student through users tags; deliveries may also apply tags when granting access. (`cademi settings tags create|delete|get|list|update`)

#### `cademi users tags create <user_id> <tag_id>`

Assign a tag · `POST /api/v3/users/{user_id}/tags/{tag_id}` · permission `users.update`

Assigns a tag to the user. The operation is idempotent: the response status indicates whether the tag was newly assigned or was already assigned.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_7`

**Flag sets:** output, idempotency

**Examples:**

```bash
# run
cademi users tags create usr_42 tag_7 --json
```

#### `cademi users tags delete <user_id> <tag_id>`

Remove a tag · `DELETE /api/v3/users/{user_id}/tags/{tag_id}` · permission `users.update`

Removes a tag from the user. The operation is idempotent: removing a tag that is not assigned also succeeds.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `tag_id` — Public ID of the tag, prefixed with `tag_`. Example: `tag_7`

**Flag sets:** output, idempotency, confirm

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users tags delete usr_42 tag_7 --yes
```

#### `cademi users tags list <user_id>`

List user tags · `GET /api/v3/users/{user_id}/tags` · permission `users.read`

Returns the tags currently assigned to the user.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users tags list usr_42 --json
```

#### `cademi users tags update <user_id>`

Replace user tags · `PUT /api/v3/users/{user_id}/tags` · permission `users.update`

Replaces the user's entire set of tags with the tags supplied in the request. Assigned tags that are not included are removed.

If any supplied tag does not exist in the account, the API returns `404 Not Found` and no changes are made.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `tags` | array of string | yes |  |

Full schema: `cademi commands users tags update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users tags update usr_42 -f 'tags[]=<value>' --json

# full body from a file
cademi users tags update usr_42 --data @body.json --json
```

### `cademi users term-acceptances` — List a user's legal term acceptances

Related settings:
- `cademi settings legal-terms` — Define legal terms here. A student's recorded acceptances are available through users term-acceptances. (`cademi settings legal-terms create|get|list|update`)

#### `cademi users term-acceptances list <user_id>`

List a user's legal term acceptances · `GET /api/v3/users/{user_id}/term-acceptances` · permission `legal_terms.read`

Lists a user's acceptances across the platform legal term and all product legal terms, using cursor-based pagination. The collection can be filtered by acceptance date.

Each item includes `links.self`, which points to the acceptance under its legal term. Use that URL with the retrieve acceptance operation.

The `proof` object is included only when the credentials have the `legal_terms.read_proof` permission.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output-basic

**Flags:**
- `--accepted-after <string>` — Only items accepted after this date and time (ISO 8601).
- `--accepted-before <string>` — Only items accepted before this date and time (ISO 8601).
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)

**Examples:**

```bash
# list: one page, machine-readable
cademi users term-acceptances list usr_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users term-acceptances list usr_42 --limit 200 --raw --json

# every page, projected
cademi users term-acceptances list usr_42 --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
