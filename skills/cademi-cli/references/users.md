---
name: cademi-cli-users
description: "Manage students, enrollments and learning progress — `cademi users` (9 commands)"
metadata:
  cademi-cli: "0.2.2"
  cademi-api: "3.10.1"
---

# Users Commands

> cademi 0.2.2, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

Users are students in the platform. Enrollments grant access through a
delivery. Products access reports effective access, sources and overrides.
Tags and custom-fields here hold a user's values; definitions are in settings.

**Related:** `cademi sales deliveries`

**Related settings:**
- `cademi settings registration` — Controls free sign-up channels, the delivery granted to new students, allowed email domains and document requirements. (`cademi settings registration get|update`)
- `cademi settings user-profile` — Controls which profile fields students can see or edit and how student names appear in support tools. (`cademi settings user-profile get|update`)
- `cademi settings tags` — Create and edit tag definitions here. Assign tags to a student through users tags; deliveries may also apply tags when granting access. (`cademi settings tags create|delete|get|list|update`)
- `cademi settings custom-fields` — Manage field definitions and types. Read or update a student's values through users custom-fields. (`cademi settings custom-fields create|delete|get|list|update`)

**See also in this domain:** `references/users-learning.md`, `references/users-profile.md`, `references/users-imports.md`, `references/users-products.md`

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

**async**
- `--wait` — When the API answers 202, wait for the operation to finish
- `--wait-timeout <duration>` — Maximum time for --wait, e.g. 30s or 2m (default: no limit; does not cancel the operation)

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi users create`

Create a user · `POST /api/v3/users` · permission `users.create`

Creates a user. Tags and custom field values supplied in the request are applied to the new user.

The `Location` header of the response contains the URL of the new user.

If another user in the account already has the same email address or `external_id`, the API returns `already_exists`. If the account has reached the user limit of its plan, the API returns `plan_limit_reached`.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `custom_fields` | object |  | Map of custom field public IDs (cfd_...) to values, not a list of field definitions. Discover IDs, labels and types with GET /settings/user-profile/custom-fields. Prefer string values: text as text, numbers as numeric strings and dates as… |
| `document` | string, nullable |  |  |
| `email` | string | yes |  |
| `external_id` | string, nullable |  |  |
| `name` | string | yes |  |
| `phone` | string, nullable |  |  |
| `send_credentials` | boolean |  |  |
| `tags` | array of string |  |  |

Full schema: `cademi commands users create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users create -f email=<email> -f name=<name> --json

# full body from a file
cademi users create --data @body.json --json
```

### `cademi users delete <user_id>`

Delete a user · `DELETE /api/v3/users/{user_id}` · permission `users.delete`

Moves the user to the trash. The user's enrollments, access, and associations are preserved.

To restore the user, send `deleted: false` to the update operation.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, idempotency, confirm

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users delete usr_42 --yes
```

### `cademi users get <user_id>`

Retrieve a user · `GET /api/v3/users/{user_id}` · permission `users.read`

Retrieves a user by public ID, including the user's tags and current `revision`.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the user to avoid overwriting a newer version.

The `document` and `phone` fields are returned only when the credentials have the `users.read_personal` permission; otherwise, they are omitted.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users get usr_42 --json
```

### `cademi users list`

List users · `GET /api/v3/users` · permission `users.read`

Returns the users of the account, using cursor-based pagination.

The collection can be filtered by a search term matched against name and email (`q`), by `external_id`, by tag (`tag_id[]`, which matches users with any of the given tags), by `status` (`active` means the user accessed the platform in the last 30 days), by `access` (the same value returned in each user), and by creation date range (`created_after`, `created_before`). Users in the trash are excluded by default; set `deleted=true` to list only users in the trash. Filtering by `access=none` without `deleted` lists the users in the trash.

The `document` and `phone` fields are returned only when the credentials have the `users.read_personal` permission.

**Flag sets:** output-basic

**Flags:**
- `--access <string>` — Platform access state: granted means outside the trash; none means in the trash. This does not test product enrollments or effective product access. Without an explicit deleted filter, access=none selects the trash. (granted, none)
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--created-after <string>` — Only items created after this date and time (ISO 8601).
- `--created-before <string>` — Only items created before this date and time (ISO 8601).
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — True selects only users in the trash; false selects only users outside it. When omitted, defaults to true for access=none and false otherwise. If both filters are explicit, both apply; contradictory values return no users.
- `--external-id <string>` — Only users with this external ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200; API default: 50
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--status <string>` — Only users with this activity status. 'active' means the user accessed the platform in the last 30 days. (active, inactive)
- `--tag-id <stringSlice>` — Only users with any of these tags, by public ID. Repeat the parameter to send several tags.

**Examples:**

```bash
# list: one page, machine-readable
cademi users list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users list --limit 200 --raw --json

# every page, projected
cademi users list --all --jq '[.[] | {id}]'
```

### `cademi users update <user_id>`

Update a user · `PATCH /api/v3/users/{user_id}` · permission `users.update`

Updates a user. Only the fields supplied in the request are changed. When `settings` is supplied, any setting omitted from it is set to `false`.

To restore a user from the trash, send `deleted: false`.

`status` is derived from the user's activity and is read-only. Including it in the request returns `unknown_field`.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `document` | string, nullable |  |  |
| `email` | string |  |  |
| `external_id` | string, nullable |  |  |
| `name` | string |  |  |
| `phone` | string, nullable |  |  |
| `settings` | object |  | Communication restrictions for this student. Omitting settings preserves all three flags. Sending an object replaces all three: omitted children become false, and an empty object clears all restrictions. Neither the object nor its boolean… |
| `settings.block_comments` | boolean |  | When true, prevents this student from posting lesson comments. False removes this student-specific restriction; account and product channel settings still apply. |
| `settings.block_questions` | boolean |  | When true, prevents this student from sending questions about product content. These are support questions, not exam questions. False removes this student-specific restriction. |
| `settings.block_support` | boolean |  | When true, prevents this student from using the support ticket channel. False removes this student-specific restriction; account support settings still apply. |

Full schema: `cademi commands users update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi users update usr_42 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi users update usr_42 --data @body.json --json
```

### `cademi users access` — Update access duration for all products

#### `cademi users access update <user_id>`

Update access duration for all products · `PATCH /api/v3/users/{user_id}/access` · permission `enrollments.manage_access`

Applies the same access duration to every product the user has access to.

The request is processed asynchronously. The response returns an operation, and the `Location` header points to it so you can track its progress.

An `Idempotency-Key` header is required.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency, async

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `duration` | object | yes |  |
| `duration.type` | string | yes |  |
| `duration.value` | integer, nullable |  |  |

Full schema: `cademi commands users access update --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users access update usr_42 -F 'duration={...}' -f duration.type=<duration.type> --wait --json

# full body from a file
cademi users access update usr_42 --data @body.json --wait --json
```

### `cademi users access-emails` — Send an access email

Related settings:
- `cademi settings emails templates` — Controls whether a transactional email is sent, its access credentials and its editable message body. (`cademi settings emails templates get|list|update`)

#### `cademi users access-emails create <user_id>`

Send an access email · `POST /api/v3/users/{user_id}/access-emails` · permission `users.send_access`

Generates a new password for the user and emails the access credentials. The user's previous password stops working.

An access email can be sent to the same user at most once every 10 minutes. Earlier attempts return `email_recently_sent`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, idempotency

**Examples:**

```bash
# run
cademi users access-emails create usr_42 --json
```

### `cademi users activity` — List user activity

#### `cademi users activity list <user_id>`

List user activity · `GET /api/v3/users/{user_id}/activity` · permission `users.read`

Returns the user's 50 most recent activity entries, newest first. IP addresses are not included in the entry context.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users activity list usr_42 --json
```

### `cademi users password-reset-emails` — Send a password reset email

Related settings:
- `cademi settings emails templates` — Controls whether a transactional email is sent, its access credentials and its editable message body. (`cademi settings emails templates get|list|update`)

#### `cademi users password-reset-emails create <user_id>`

Send a password reset email · `POST /api/v3/users/{user_id}/password-reset-emails` · permission `users.send_access`

Sends the user an email with a link to reset their password.

A password reset email can be sent to the same user at most once every 10 minutes. Earlier attempts return `email_recently_sent`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, idempotency

**Examples:**

```bash
# run
cademi users password-reset-emails create usr_42 --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
