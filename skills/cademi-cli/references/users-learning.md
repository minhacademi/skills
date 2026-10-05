---
name: cademi-cli-users-learning
description: "Enrollments, progress, certificates and scores of a user — `cademi users` (13 commands)"
metadata:
  cademi-cli: "0.2.0"
  cademi-api: "3.10.0"
---

# Users Learning Commands

> cademi 0.2.0, API 3.10.0. The live catalog is always `cademi commands <prefix> --json`.

Enrollments, progress, certificates and scores of a user. The rest of `cademi users` is in the sibling files below.

**Related:** `cademi sales deliveries`

**Related settings:**
- `cademi settings registration` — Controls free sign-up channels, the delivery granted to new students, allowed email domains and document requirements. (`cademi settings registration get|update`)
- `cademi settings user-profile` — Controls which profile fields students can see or edit and how student names appear in support tools. (`cademi settings user-profile get|update`)
- `cademi settings tags` — Create and edit tag definitions here. Assign tags to a student through users tags; deliveries may also apply tags when granting access. (`cademi settings tags create|delete|get|list|update`)
- `cademi settings custom-fields` — Manage field definitions and types. Read or update a student's values through users custom-fields. (`cademi settings custom-fields create|delete|get|list|update`)

**See also in this domain:** `references/users-profile.md`, `references/users.md`, `references/users-imports.md`, `references/users-products.md`

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

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
