---
name: cademi-cli-users
description: "Manage students, enrollments and learning progress — `cademi users` (39 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Users Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Users are students in the platform. Enrollments grant access through a
delivery. Products access reports effective access, sources and overrides.
Tags and custom-fields here hold a user's values; definitions are in settings.

**Related:** `cademi sales deliveries`

**Related settings:**
- `cademi settings registration` — Controls free sign-up channels, the delivery granted to new students, allowed email domains and document requirements. (`cademi settings registration get|update`)
- `cademi settings user-profile` — Controls which profile fields students can see or edit and how student names appear in support tools. (`cademi settings user-profile get|update`)
- `cademi settings tags` — Create and edit tag definitions here. Assign tags to a student through users tags; deliveries may also apply tags when granting access. (`cademi settings tags create|delete|get|list|update`)
- `cademi settings custom-fields` — Manage field definitions and types. Read or update a student's values through users custom-fields. (`cademi settings custom-fields create|delete|get|list|update`)

**See also in this domain:** `references/users-imports.md`, `references/users-products.md`

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

### `cademi users certificates` — Manage certificates issued to students

Inspect and manage issued certificates. Templates and previews belong to
content products certificate; aggregated results belong to reports certificates.

Related: `cademi content products certificate`, `cademi reports certificates`

#### `cademi users certificates create <user_id>`

Issue a certificate · `POST /api/v3/users/{user_id}/certificates` · permission `certificates.create`

Issues a certificate to the user for the specified product.

If the user already holds a valid certificate for the product, the existing certificate is returned instead of a new one being issued.

To reissue a certificate, set `supersedes_certificate_id` to the certificate being replaced. Reissuing requires the `certificates.reissue` permission in addition to `certificates.create`, and the certificate identified in `supersedes_certificate_id` is revoked with the reason `reissued`. That certificate must belong to the same user and product; otherwise, the request is rejected with a validation error on `supersedes_certificate_id`. A certificate that is already revoked cannot be superseded.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `product_id` | string | yes |  |
| `supersedes_certificate_id` | string, nullable |  |  |

Full schema: `cademi commands users certificates create --schema --json`

Legacy path: `cademi certificates create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users certificates create usr_42 -f product_id=<product_id> --json

# full body from a file
cademi users certificates create usr_42 --data @body.json --json
```

#### `cademi users certificates delete <user_id> <certificate_id>`

Delete a certificate · `DELETE /api/v3/users/{user_id}/certificates/{certificate_id}` · permission `certificates.delete`

Moves the certificate to the trash. A `reason` is required and is recorded with the deletion.

Deleted certificates no longer appear in the user's certificate list and can no longer be verified through public validation. If the user is still eligible, they can obtain a new certificate on demand. To invalidate a certificate while keeping it on record, revoke it through the update operation instead.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `certificate_id` — Public ID of the certificate, prefixed with `cer_`. Example: `cer_9`

**Flag sets:** output, body, idempotency, confirm

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | yes |  |

Full schema: `cademi commands users certificates delete --schema --json`

Legacy path: `cademi certificates delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi users certificates delete usr_42 cer_9 --yes
```

#### `cademi users certificates get <user_id> <certificate_id>`

Retrieve a certificate · `GET /api/v3/users/{user_id}/certificates/{certificate_id}` · permission `certificates.read`

Revoked certificates remain retrievable, with `status` set to `revoked`. Template previews are not certificates and cannot be retrieved through this operation.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when revoking the certificate to avoid acting on an outdated version.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `certificate_id` — Public ID of the certificate, prefixed with `cer_`. Example: `cer_9`

**Flag sets:** output

Legacy path: `cademi certificates get`

**Examples:**

```bash
# get
cademi users certificates get usr_42 cer_9 --json
```

#### `cademi users certificates list <user_id>`

List a user's certificates · `GET /api/v3/users/{user_id}/certificates` · permission `certificates.read`

Returns the certificates issued to the user, including revoked certificates. Deleted certificates are not included.

The collection can be filtered by product, status, and issue date, and is paginated with a cursor. When the credentials are restricted to specific products, only certificates for those products are returned.

The `fields` object, which contains the document, address, and custom field values recorded at issuance, is included only when the credential has the `users.read_personal` permission.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--issued-after <string>` — Only items issued after this date and time (ISO 8601).
- `--issued-before <string>` — Only items issued before this date and time (ISO 8601).
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--product-id <string>` — Only certificates issued for the product with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only certificates with this status. (valid, revoked)

Legacy path: `cademi certificates list`

**Examples:**

```bash
# list: one page, machine-readable
cademi users certificates list usr_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users certificates list usr_42 --limit 200 --raw --json

# every page, projected
cademi users certificates list usr_42 --all --jq '[.[] | {id}]'
```

#### `cademi users certificates update <user_id> <certificate_id>`

Revoke a certificate · `PATCH /api/v3/users/{user_id}/certificates/{certificate_id}` · permission `certificates.revoke`

Revokes a certificate. Revocation is the only supported change: set `status` to `revoked` and provide a `reason`. Any other field is rejected.

A revoked certificate remains on record and retrievable, and public validation reports it as revoked. Revocation does not affect the user's access, progress, or points. To remove a certificate, use the delete operation instead.

Send the certificate's current `ETag` in the `If-Match` header to avoid acting on an outdated version.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `certificate_id` — Public ID of the certificate, prefixed with `cer_`. Example: `cer_9`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | yes | Why the certificate is revoked. It is stored and sent as written in the `certificate.revoked` event, which webhooks receive at every payload detail level, including `ids`. Do not include personal data. |
| `status` | string | yes | One of: `revoked` |

Full schema: `cademi commands users certificates update --schema --json`

Legacy path: `cademi certificates update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users certificates update usr_42 cer_9 -f reason=<reason> -f status=revoked --json

# full body from a file
cademi users certificates update usr_42 cer_9 --data @body.json --json
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

### `cademi users enrollments` — Grant and manage a user's access through deliveries

Supply a user ID. Creating an enrollment requires a delivery_id in the body.
The delivery determines covered products and access duration. Effective
access also depends on release rules and user overrides. Revocation and
restoration are limited to manually granted enrollments.

Related: `cademi sales deliveries`, `cademi users products access`, `cademi reports enrollments`

```bash
cademi users enrollments list usr_42
cademi users enrollments create usr_42 -f delivery_id=dlv_7
```

#### `cademi users enrollments create <user_id>`

Create an enrollment · `POST /api/v3/users/{user_id}/enrollments` · permission `enrollments.create`

Grants the user access to a delivery. The `Idempotency-Key` header is required.

If the user already has an enrollment for the same delivery, the existing enrollment is reactivated and returned with `200 OK` instead of creating a new one.

The credentials must have access to every product included in the delivery. A delivery that does not exist, is archived, has no products, or includes products outside the scope of the current credentials is rejected with a validation error on `delivery_id`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `delivery_id` | string | yes |  |
| `sale_id` | string, nullable |  |  |
| `subscription_id` | string, nullable |  |  |

Full schema: `cademi commands users enrollments create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users enrollments create usr_42 -f delivery_id=<delivery_id> --json

# full body from a file
cademi users enrollments create usr_42 --data @body.json --json
```

#### `cademi users enrollments get <user_id> <enrollment_id>`

Retrieve an enrollment · `GET /api/v3/users/{user_id}/enrollments/{enrollment_id}` · permission `enrollments.read`

Retrieves an enrollment of the user by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the enrollment to avoid overwriting a newer version.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `enrollment_id` — Public ID of the enrollment, prefixed with `enr_`. Example: `enr_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users enrollments get usr_42 enr_42 --json
```

#### `cademi users enrollments list <user_id>`

List a user's enrollments · `GET /api/v3/users/{user_id}/enrollments` · permission `enrollments.read`

Lists the user's enrollments, one per delivery.

When the credentials are restricted to specific products, only enrollments for deliveries that include at least one of those products are returned. Revoked enrollments are excluded unless `status=revoked` or `deleted=true` is supplied.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `--delivery-id <string>` — Only enrollments granted by the delivery with this public ID.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--origin <string>` — Only enrollments with this origin. (manual, sale, subscription)
- `--product-id <string>` — Only enrollments in the product with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--status <string>` — Only enrollments with this status. (active, suspended, revoked)
- `--updated-after <string>` — Only items updated after this date and time (ISO 8601).

**Examples:**

```bash
# list: one page, machine-readable
cademi users enrollments list usr_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users enrollments list usr_42 --limit 200 --raw --json

# every page, projected
cademi users enrollments list usr_42 --all --jq '[.[] | {id}]'
```

#### `cademi users enrollments update <user_id> <enrollment_id>`

Update an enrollment · `PATCH /api/v3/users/{user_id}/enrollments/{enrollment_id}` · permission `enrollments.update`, `enrollments.delete (if field:deleted)`, `enrollments.delete (if transition:status=revoked)`

Updates the status or scheduled end of an enrollment, or restores a revoked enrollment.

Setting `status` to `revoked` or `deleted` to `false` requires the `enrollments.delete` permission in addition to `enrollments.update`. Only manually granted enrollments can be revoked or restored; enrollments originating from a sale or subscription return `state_conflict`.

A revoked enrollment only accepts `deleted: false`; changing its `status` or `ends_at` returns `state_conflict` with `reason` `revoked`. When `deleted: false` is sent together with `status` or `ends_at`, the enrollment is restored first and then updated.

`ends_at` must be an ISO 8601 date-time with a time zone offset. Send `null` to remove the scheduled end.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve operation in the `If-Match` header.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `enrollment_id` — Public ID of the enrollment, prefixed with `enr_`. Example: `enr_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  | Only `false` is accepted. Restores it from the trash; deleting is the DELETE operation. One of: `false` |
| `ends_at` | date-time, nullable |  |  |
| `status` | string |  | One of: `active`, `suspended`, `revoked` |

Full schema: `cademi commands users enrollments update --schema --json`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi users enrollments update usr_42 enr_42 -f deleted=false --if-match '"<etag>"' --json

# full body from a file
cademi users enrollments update usr_42 enr_42 --data @body.json --json
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

### `cademi users progress` — List user progress

#### `cademi users progress list <user_id>`

List user progress · `GET /api/v3/users/{user_id}/progress` · permission `progress.read`

Returns the user's progress in each product, using cursor-based pagination.

If the current credentials are restricted to specific products, only those products are included.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)

**Examples:**

```bash
# list: one page, machine-readable
cademi users progress list usr_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users progress list usr_42 --limit 200 --raw --json

# every page, projected
cademi users progress list usr_42 --all --jq '[.[] | {id}]'
```

### `cademi users score-adjustments` — Create a points adjustment

#### `cademi users score-adjustments create <user_id>`

Create a points adjustment · `POST /api/v3/users/{user_id}/score-adjustments` · permission `scores.adjust`

Creates a manual adjustment to the user's points. The adjustment is recorded as a point entry in the points ledger together with its author and `reason`, and the user's balance changes by the adjusted amount.

`points` must be a non-zero integer; negative values deduct points. Use `product_id` to associate the adjustment with a product.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `points` | integer | yes |  |
| `product_id` | string, nullable |  |  |
| `reason` | string | yes |  |

Full schema: `cademi commands users score-adjustments create --schema --json`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi users score-adjustments create usr_42 -F points=1 -f reason=<reason> --json

# full body from a file
cademi users score-adjustments create usr_42 --data @body.json --json
```

### `cademi users scores` — Manage users scores

#### `cademi users scores get <user_id> <score_id>`

Retrieve a point entry · `GET /api/v3/users/{user_id}/scores/{score_id}` · permission `scores.read`

Retrieves a point entry from the user's points ledger. The entry must belong to the user identified in the path; otherwise, the API returns `404 Not Found`.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`
- `score_id` — Public ID of the score, prefixed with `sco_`. Example: `sco_42`

**Flag sets:** output

**Examples:**

```bash
# get
cademi users scores get usr_42 sco_42 --json
```

#### `cademi users scores list <user_id>`

List point entries · `GET /api/v3/users/{user_id}/scores` · permission `scores.read`

Returns the user's points ledger, using cursor-based pagination.

If the user is not accessible with the current credentials, the API returns `404 Not Found` rather than an empty collection.

**Arguments:**
- `user_id` — Public ID of the user, prefixed with `usr_`. Example: `usr_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)

**Examples:**

```bash
# list: one page, machine-readable
cademi users scores list usr_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi users scores list usr_42 --limit 200 --raw --json

# every page, projected
cademi users scores list usr_42 --all --jq '[.[] | {id}]'
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
