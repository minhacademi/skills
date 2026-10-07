---
name: cademi-cli-account
description: "Inspect the account, administrators and available capabilities — `cademi account` (18 commands)"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Account Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

Account operations describe the platform, domains and replicas.
Administrators manage the platform; users are its students. Capabilities
describe available API features, and audit-entries record administrative activity.

**Related:** `cademi integrations credentials`

Live catalog for this file: `cademi commands account --json` (offline, no credential needed).

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

### `cademi account get`

Retrieve the account · `GET /api/v3/account` · permission `account.read`

Returns a read-only overview of the account associated with the current credentials: identity, student quota, billing status, and replica and sandbox relationships.

Billing information does not include amounts, plan details, or payment methods.

Requires the `account.read` permission.

**Flag sets:** output

**Examples:**

```bash
# get
cademi account get --json
```

### `cademi account usage`

Retrieve account usage · `GET /api/v3/account/usage` · permission `usage.read`

Returns aggregated API usage for the account over the time window selected by `period`, including request and operation totals, a breakdown by credential, and rate limit information.

`storage` reports the file storage in use and the storage quota of the account, in bytes, regardless of `period`. The quota also covers the accounts billed with it, such as its replicas, and uploads that would exceed it are rejected with `quota_exceeded`.

Request and operation figures are computed from live data when the request is made, so `ingestion_delay_seconds` is always `0`, and the response is never cached. `storage.used_bytes` comes from a stored total that is updated on each upload and deletion through the API and recalculated daily, so files added or removed in the dashboard may take up to a day to be reflected.

Requires the `usage.read` permission.

**Flag sets:** output

**Flags:**
- `--period <string>` — Time window of the aggregated figures, ending now. (1h, 24h, 7d, 30d) (required)

**Examples:**

```bash
# get
cademi account usage --json
```

### `cademi account administrators` — Manage administrators and their platform permissions

Administrators operate the dashboard and API. Students are managed under
users; API credential policies are managed under integrations credentials.

Related: `cademi integrations credentials`

#### `cademi account administrators create`

Create an administrator · `POST /api/v3/administrators` · permission `administrators.create`

Creates an administrator in the current account.

By default, the new administrator receives an access email. Set `send_credentials` to `false` to skip it.

You can assign granular permissions and product access in the same request with `permissions` and `product_ids`, which also requires the `administrators.manage_permissions` permission. When `permissions` is supplied, the administrator receives only the permissions listed there instead of the role's default permissions; `product_ids` alone keeps the role's default permissions. If any part of the request is rejected, no administrator is created.

_Full description: `cademi commands account administrators create --json --jq '.commands[0].description'`_

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | string | yes |  |
| `name` | string | yes |  |
| `permissions` | object |  |  |
| `product_ids` | array of string |  |  |
| `role` | string | yes |  |
| `send_credentials` | boolean |  |  |

Full schema: `cademi commands account administrators create --schema --json`

Legacy path: `cademi administrators create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi account administrators create -f email=<email> -f name=<name> -f role=<role> --json

# full body from a file
cademi account administrators create --data @body.json --json
```

#### `cademi account administrators delete <administrator_id>`

Delete an administrator · `DELETE /api/v3/administrators/{administrator_id}` · permission `administrators.delete`

Moves the administrator to the trash. A trashed administrator can be restored with the update operation by setting `deleted` to `false`.

The last `root` administrator in the account cannot be deleted, and in human mode administrators cannot delete themselves. Administrators with the `root` or `master_admin` role cannot be deleted through the API; attempts return `delegation_limit_exceeded`.

API credentials created by the deleted administrator are not revoked.

**Arguments:**
- `administrator_id` — Public ID of the administrator, prefixed with `adm_`. Example: `adm_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi administrators delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi account administrators delete adm_42 --yes
```

#### `cademi account administrators get <administrator_id>`

Retrieve an administrator · `GET /api/v3/administrators/{administrator_id}` · permission `administrators.read`

Retrieves an administrator by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the administrator or its permissions to avoid overwriting a newer version.

Administrators in the trash are not returned by this operation.

**Arguments:**
- `administrator_id` — Public ID of the administrator, prefixed with `adm_`. Example: `adm_42`

**Flag sets:** output

Legacy path: `cademi administrators get`

**Examples:**

```bash
# get
cademi account administrators get adm_42 --json
```

#### `cademi account administrators list`

List administrators · `GET /api/v3/administrators` · permission `administrators.read`

Lists the administrators in the current account using cursor-based pagination.

By default, only active administrators are returned. Set `deleted` to `true` to list only administrators in the trash. Use `q` to search by name or email address (case-insensitive partial match) and `role` to filter by role.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--q <string>` — Text search term.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--role <string>` — Only administrators with this role.

Legacy path: `cademi administrators list`

**Examples:**

```bash
# list: one page, machine-readable
cademi account administrators list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi account administrators list --limit 200 --raw --json

# every page, projected
cademi account administrators list --all --jq '[.[] | {id}]'
```

#### `cademi account administrators update <administrator_id>`

Update an administrator · `PATCH /api/v3/administrators/{administrator_id}` · permission `administrators.update`

Updates an administrator's name or email address, or restores an administrator from the trash.

To restore a trashed administrator, set `deleted` to `false`. Restoring requires the `administrators.delete` permission and is rejected with `validation_failed` if another active administrator already uses the same email address.

Roles and permissions can only be changed through the update administrator permissions operation. Fields such as `role`, `status`, and `password` are rejected with `unknown_field`.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `administrator_id` — Public ID of the administrator, prefixed with `adm_`. Example: `adm_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `deleted` | boolean |  |  |
| `email` | string |  |  |
| `name` | string |  |  |

Full schema: `cademi commands account administrators update --schema --json`

Legacy path: `cademi administrators update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi account administrators update adm_42 -F deleted=true --if-match '"<etag>"' --json

# full body from a file
cademi account administrators update adm_42 --data @body.json --json
```

### `cademi account administrators invitation-emails` — Resend the access email

Related: `cademi integrations credentials`

#### `cademi account administrators invitation-emails create <administrator_id>`

Resend the access email · `POST /api/v3/administrators/{administrator_id}/invitation-emails` · permission `administrators.send_access`

Resends the access email to the administrator.

This operation replaces the administrator's password with a new one, which is sent by email and never included in the response. The email is sent asynchronously.

An access email can be sent to the same administrator at most once every 10 minutes. Earlier attempts return `email_recently_sent`.

**Arguments:**
- `administrator_id` — Public ID of the administrator, prefixed with `adm_`. Example: `adm_42`

**Flag sets:** output, idempotency

Legacy path: `cademi administrators invitation-emails create`

**Examples:**

```bash
# run
cademi account administrators invitation-emails create adm_42 --json
```

### `cademi account administrators permissions` — Manage account administrators permissions

Related: `cademi integrations credentials`

#### `cademi account administrators permissions get <administrator_id>`

Retrieve administrator permissions · `GET /api/v3/administrators/{administrator_id}/permissions` · permission `administrators.read`

Returns the administrator's role, the full set of granular permissions with the state of each one, and the IDs of the products the administrator can access.

**Arguments:**
- `administrator_id` — Public ID of the administrator, prefixed with `adm_`. Example: `adm_42`

**Flag sets:** output

Legacy path: `cademi administrators permissions get`

**Examples:**

```bash
# get
cademi account administrators permissions get adm_42 --json
```

#### `cademi account administrators permissions update <administrator_id>`

Update administrator permissions · `PUT /api/v3/administrators/{administrator_id}/permissions` · permission `administrators.manage_permissions`

Replaces the administrator's role, granular permissions, and product access in a single operation. Permissions not granted in `permissions` are disabled, and product access is set to exactly the products listed in `product_ids`. If any part of the request is rejected, no changes are applied.

Permission keys that do not exist in the permission catalog return `validation_failed`.

The API cannot assign the `root` or `master_admin` role, and every permission granted must be part of the selected role's default permissions. Requests that exceed these limits return `delegation_limit_exceeded`.

To avoid overwriting a newer version, send the `ETag` returned by the retrieve administrator operation in the `If-Match` header. The response includes the new `ETag`.

**Arguments:**
- `administrator_id` — Public ID of the administrator, prefixed with `adm_`. Example: `adm_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `permissions` | object | yes |  |
| `product_ids` | array of string |  |  |
| `role` | string | yes |  |

Full schema: `cademi commands account administrators permissions update --schema --json`

Legacy path: `cademi administrators permissions update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi account administrators permissions update adm_42 -F 'permissions={...}' -f role=<role> --json

# full body from a file
cademi account administrators permissions update adm_42 --data @body.json --json
```

### `cademi account audit-entries` — Inspect the administrative audit trail

Inspect recorded administrative changes and their authors.

Related: `cademi integrations requests`

#### `cademi account audit-entries get <audit_entry_id>`

Retrieve an audit entry · `GET /api/v3/audit-entries/{audit_entry_id}` · permission `audit.read`

Retrieves an audit entry by its public ID, which uses the `aud_` prefix (for example, `aud_42`).

Malformed IDs and entries that do not exist or are not accessible with the current credentials return `404 Not Found`.

**Arguments:**
- `audit_entry_id` — Public ID of the audit entry, prefixed with `aud_`. Example: `aud_42`

**Flag sets:** output

Legacy path: `cademi audit-entries get`

**Examples:**

```bash
# get
cademi account audit-entries get aud_42 --json
```

#### `cademi account audit-entries list`

List audit entries · `GET /api/v3/audit-entries` · permission `audit.read`

Lists the audit entries of the current account: write operations, denied or rejected requests, background effects, and management actions. Successful read requests are not included; they are available through the Requests endpoints.

Results are paginated by cursor and sorted by `occurred_at`, newest first by default. The collection can be filtered by kind, outcome, action code, resource, request ID, and occurrence time range.

Requires the `audit.read` permission for the account.

**Flag sets:** output-basic

**Flags:**
- `--action-code <string>` — Only entries with this action code.
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--kind <stringSlice>` — Only entries of these kinds. Repeat the parameter to match any of several kinds.
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--occurred-after <string>` — Only items that occurred after this date and time (ISO 8601).
- `--occurred-before <string>` — Only items that occurred before this date and time (ISO 8601).
- `--outcome <stringSlice>` — Only entries with these outcomes. Repeat the parameter to match any of several outcomes.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--request-id <string>` — Only entries recorded by the request with this ID, as returned in 'X-Request-Id'.
- `--resource-id <string>` — Only entries about the resource with this public ID.
- `--resource-type <string>` — Only entries about resources of this type.
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (occurred_at, -occurred_at)

Legacy path: `cademi audit-entries list`

**Examples:**

```bash
# list: one page, machine-readable
cademi account audit-entries list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi account audit-entries list --limit 200 --raw --json

# every page, projected
cademi account audit-entries list --all --jq '[.[] | {id}]'
```

### `cademi account capabilities` — Discover features available through the API

Inspect capabilities before using features that may vary by account.

Related: `cademi doctor`

#### `cademi account capabilities get`

Retrieve credential capabilities · `GET /api/v3/capabilities`

Returns the permissions currently in effect for the authenticated credential, together with the permission catalog version, the environment, and the rate limits that apply to the credential.

Any authenticated credential can call this operation; no specific permission is required. The call is read-only and does not grant or change any permissions.

**Flag sets:** output

Legacy path: `cademi capabilities get`

**Examples:**

```bash
# get
cademi account capabilities get --json
```

### `cademi account domains` — Manage account domains

#### `cademi account domains get <domain_id>`

Retrieve a domain · `GET /api/v3/account/domains/{domain_id}` · permission `domains.read`

Retrieves the account's custom domain by its public ID.

An account has at most one custom domain, and its ID is fixed for the account. Use the ID returned by the list operation. Any other ID returns `404 Not Found`.

**Arguments:**
- `domain_id` — Public ID of the domain, prefixed with `dom_`. Example: `dom_1`

**Flag sets:** output

**Examples:**

```bash
# get
cademi account domains get dom_1 --json
```

#### `cademi account domains list`

List domains · `GET /api/v3/account/domains` · permission `domains.read`

Lists the custom domains configured for the account.

An account has at most one custom domain, so the collection contains zero or one item. Domain status reflects the last state recorded by Cademí and is not checked live.

Requires the `domains.read` permission.

**Flag sets:** output

**Examples:**

```bash
# get
cademi account domains list --json
```

### `cademi account domains certificate` — Retrieve a domain certificate

#### `cademi account domains certificate get <domain_id>`

Retrieve a domain certificate · `GET /api/v3/account/domains/{domain_id}/certificate` · permission `domains.read`

Returns the TLS certificate status of the account's custom domain.

The status reflects the last state recorded by Cademí and is not checked live with the certificate provider.

**Arguments:**
- `domain_id` — Public ID of the domain, prefixed with `dom_`. Example: `dom_1`

**Flag sets:** output

**Examples:**

```bash
# get
cademi account domains certificate get dom_1 --json
```

### `cademi account replicas` — Manage account replicas

#### `cademi account replicas get <replica_id>`

Retrieve a replica · `GET /api/v3/account/replicas/{replica_id}` · permission `replicas.read`

Retrieves a replica of the current account by its public ID.

Only replicas of the account associated with the current credentials can be retrieved. Accounts that only share billing with the current account are not replicas. Any other ID, including the current account's own, returns `404 Not Found`.

**Arguments:**
- `replica_id` — Public ID of the replica, prefixed with `rep_`. Example: `rep_1`

**Flag sets:** output

**Examples:**

```bash
# get
cademi account replicas get rep_1 --json
```

#### `cademi account replicas list`

List replicas · `GET /api/v3/account/replicas` · permission `replicas.read`

Lists the replicas of the account associated with the current credentials, sorted by ID and paginated with a cursor.

Accounts that only share billing with the current account are not replicas and are not included.

Requires the `replicas.read` permission, which grants access to every replica of the account.

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
cademi account replicas list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi account replicas list --limit 200 --raw --json

# every page, projected
cademi account replicas list --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
