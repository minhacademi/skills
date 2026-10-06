---
name: cademi-cli-integrations-credentials
description: "Manage API credentials and their access policies — `cademi integrations credentials` (15 commands)"
metadata:
  cademi-cli: "0.2.3"
  cademi-api: "3.10.1"
---

# Integrations Credentials Commands

> cademi 0.2.3, API 3.10.1. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi integrations` — Manage API credentials, events and webhook delivery.

Inspect the current credential or manage credentials and resource policies.
CLI login and locally saved connections are managed by auth and profiles.

**Related:** `cademi auth`, `cademi profiles`, `cademi account capabilities`

**See also in this domain:** `references/integrations.md`, `references/integrations-webhooks.md`

Live catalog for this file: `cademi commands integrations credentials --json` (offline, no credential needed).

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

### `cademi integrations credentials create`

Create a credential · `POST /api/v3/credentials` · permission `credentials.manage`

Issues a new API credential for the current account and, optionally, its initial access policies.

Each entry in `policies` specifies either a list of `capabilities` or a policy `template`, together with the `resources` it applies to. Available templates are returned by the list policy templates operation.

The credential secret is returned only in this response and cannot be retrieved again. Store it securely. Retrying the request with the same `Idempotency-Key` does not return the secret again; it returns `409` with the `secret_not_replayable` error code.

The new credential cannot receive permissions beyond those the calling credential is allowed to delegate. Such requests are rejected with `delegation_limit_exceeded` and no credential is issued.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `auth_mode` | string |  | One of: `autonomous`, `human_required`, `both` |
| `expires_at` | date-time, nullable |  |  |
| `name` | string | yes |  |
| `policies` | array of object |  |  |
| `purpose` | string, nullable |  |  |

Full schema: `cademi commands integrations credentials create --schema --json`

Legacy path: `cademi credentials create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi integrations credentials create -f name=<name> --json

# full body from a file
cademi integrations credentials create --data @body.json --json
```

### `cademi integrations credentials current`

Retrieve the current credential · `GET /api/v3/credentials/current`

Returns the credential used to authenticate the request, reflecting its current state. Available to any authenticated credential without additional permissions.

**Flag sets:** output

Legacy path: `cademi credentials current`

**Examples:**

```bash
# get
cademi integrations credentials current --json
```

### `cademi integrations credentials current-policies`

Retrieve the current credential's policies · `GET /api/v3/credentials/current/policies`

Returns the active policy revision of the credential used to authenticate the request. Use this operation to inspect what the calling credential is allowed to do; it cannot be used to read other credentials.

**Flag sets:** output

Legacy path: `cademi credentials current-policies`

**Examples:**

```bash
# get
cademi integrations credentials current-policies --json
```

### `cademi integrations credentials delete <credential_id>`

Delete a credential · `DELETE /api/v3/credentials/{credential_id}` · permission `credentials.manage`

Deletes a revoked or expired credential. After deletion, the credential is no longer returned by the API, while audit records that reference it are preserved.

Credentials that are still valid cannot be deleted and return the `state_conflict` error code. Revoke the credential with the update operation first.

A credential cannot delete itself.

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi credentials delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi integrations credentials delete key_01J8Z3 --yes
```

### `cademi integrations credentials get <credential_id>`

Retrieve a credential · `GET /api/v3/credentials/{credential_id}` · permission `credentials.read`

Retrieves a credential by its public ID.

The `secret` object exposes only metadata, such as the last four characters and the rotation date. The secret value itself is never returned by this operation.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the credential to avoid overwriting a newer version.

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`

**Flag sets:** output

Legacy path: `cademi credentials get`

**Examples:**

```bash
# get
cademi integrations credentials get key_01J8Z3 --json
```

### `cademi integrations credentials list`

List credentials · `GET /api/v3/credentials` · permission `credentials.read`

Returns all credentials of the current account, including the calling credential. Requires the `credentials.read` permission; resource-level restrictions do not apply to this collection.

Results are paginated with a cursor and can be filtered by `status` and sorted by creation date.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (created_at, -created_at)
- `--status <string>` — Only credentials with this status. (active, suspended, revoked)

Legacy path: `cademi credentials list`

**Examples:**

```bash
# list: one page, machine-readable
cademi integrations credentials list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi integrations credentials list --limit 200 --raw --json

# every page, projected
cademi integrations credentials list --all --jq '[.[] | {id}]'
```

### `cademi integrations credentials update <credential_id>`

Update a credential · `PATCH /api/v3/credentials/{credential_id}` · permission `credentials.manage`

Updates the `name`, `purpose`, or `expires_at` of a credential, or changes its `status`.

Setting `status` to `suspended` suspends the credential, and setting it to `active` reactivates it. `inactive` is a deprecated alias of `suspended`. Setting `status` to `revoked` permanently revokes the credential; revocation cannot be undone.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

A credential cannot update itself.

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `expires_at` | date-time, nullable |  |  |
| `name` | string |  |  |
| `purpose` | string, nullable |  |  |
| `status` | string |  | `inactive` is a deprecated alias of `suspended`. One of: `active`, `suspended`, `revoked`, `inactive` |

Full schema: `cademi commands integrations credentials update --schema --json`

Legacy path: `cademi credentials update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi integrations credentials update key_01J8Z3 -f expires_at=<expires_at> --if-match '"<etag>"' --json

# full body from a file
cademi integrations credentials update key_01J8Z3 --data @body.json --json
```

### `cademi integrations credentials permission-catalog` — Retrieve the permission catalog

Related: `cademi auth`, `cademi profiles`, `cademi account capabilities`

#### `cademi integrations credentials permission-catalog get`

Retrieve the permission catalog · `GET /api/v3/credentials/permission-catalog`

Returns every permission supported by the API, with its code, resource type, description, whether it can be delegated to other credentials, and the API version in which it was introduced.

Available to any authenticated credential. Reading the catalog does not grant any permission.

**Flag sets:** output

Legacy path: `cademi credentials permission-catalog get`

**Examples:**

```bash
# get
cademi integrations credentials permission-catalog get --json
```

### `cademi integrations credentials policies` — Manage integrations credentials policies

Related: `cademi auth`, `cademi profiles`, `cademi account capabilities`

#### `cademi integrations credentials policies create <credential_id>`

Add a credential policy · `POST /api/v3/credentials/{credential_id}/policies` · permission `credentials.policies.manage`

Adds a policy to a credential. The change publishes a new policy revision containing the existing policies plus the new one.

Permissions listed in `capabilities` that the calling credential cannot delegate, including the non-delegable `credentials.*` and `administrators.*` permissions, are rejected with `delegation_limit_exceeded`, and no new revision is published.

A credential cannot modify its own policies, and the policies of a revoked credential cannot be changed (`credential_revoked`).

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `capabilities` | array of string | yes |  |
| `resources` | array of object | yes |  |

Full schema: `cademi commands integrations credentials policies create --schema --json`

Legacy path: `cademi credentials policies create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi integrations credentials policies create key_01J8Z3 -f 'capabilities[]=<value>' --data '{"resources":[...]}' --json

# full body from a file
cademi integrations credentials policies create key_01J8Z3 --data @body.json --json
```

#### `cademi integrations credentials policies delete <credential_id> <policy_id>`

Remove a credential policy · `DELETE /api/v3/credentials/{credential_id}/policies/{policy_id}` · permission `credentials.policies.manage`

Removes a policy from a credential. The change publishes a new policy revision containing all remaining policies.

Removing the last policy is allowed and leaves the credential without access to any resource.

A credential cannot modify its own policies, and the policies of a revoked credential cannot be changed (`credential_revoked`).

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`
- `policy_id` — Public ID of the policy, prefixed with `pol_`. Example: `pol_01J8Z3`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi credentials policies delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi integrations credentials policies delete key_01J8Z3 pol_01J8Z3 --yes
```

#### `cademi integrations credentials policies get <credential_id> <policy_id>`

Retrieve a credential policy · `GET /api/v3/credentials/{credential_id}/policies/{policy_id}` · permission `credentials.read`

Retrieves a policy from the credential's active policy revision.

Policies that are no longer part of the active revision, or that belong to a different credential, return `404 Not Found`.

The response includes an `ETag` representing the credential's current revision. Send this value in the `If-Match` header when updating the policy to avoid overwriting a newer version.

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`
- `policy_id` — Public ID of the policy, prefixed with `pol_`. Example: `pol_01J8Z3`

**Flag sets:** output

Legacy path: `cademi credentials policies get`

**Examples:**

```bash
# get
cademi integrations credentials policies get key_01J8Z3 pol_01J8Z3 --json
```

#### `cademi integrations credentials policies list <credential_id>`

List credential policies · `GET /api/v3/credentials/{credential_id}/policies` · permission `credentials.read`

Returns the policies in the credential's active policy revision, one item per policy.

A credential without policies returns an empty collection.

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`

**Flag sets:** output

Legacy path: `cademi credentials policies list`

**Examples:**

```bash
# get
cademi integrations credentials policies list key_01J8Z3 --json
```

#### `cademi integrations credentials policies update <credential_id> <policy_id>`

Update a credential policy · `PATCH /api/v3/credentials/{credential_id}/policies/{policy_id}` · permission `credentials.policies.manage`

Replaces the capabilities and resources of a policy. The change publishes a new policy revision in which the other policies remain unchanged. The policy keeps the same `policy_id` across revisions.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version.

Permissions that the calling credential cannot delegate are rejected with `delegation_limit_exceeded`, and no new revision is published. A credential cannot modify its own policies, and the policies of a revoked credential cannot be changed (`credential_revoked`).

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`
- `policy_id` — Public ID of the policy, prefixed with `pol_`. Example: `pol_01J8Z3`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `capabilities` | array of string | yes |  |
| `resources` | array of object | yes |  |

Full schema: `cademi commands integrations credentials policies update --schema --json`

Legacy path: `cademi credentials policies update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi integrations credentials policies update key_01J8Z3 pol_01J8Z3 -f 'capabilities[]=<value>' --data '{"resources":[...]}' --json

# full body from a file
cademi integrations credentials policies update key_01J8Z3 pol_01J8Z3 --data @body.json --json
```

### `cademi integrations credentials policy-templates` — List policy templates

Related: `cademi auth`, `cademi profiles`, `cademi account capabilities`

#### `cademi integrations credentials policy-templates list`

List policy templates · `GET /api/v3/credentials/policy-templates`

Returns the predefined policy templates that can be used when creating a credential, each with the list of permissions it grants. No template includes `administrators.*` or `credentials.*` permissions.

Available to any authenticated credential. Reading the templates does not grant any permission.

**Flag sets:** output

Legacy path: `cademi credentials policy-templates list`

**Examples:**

```bash
# get
cademi integrations credentials policy-templates list --json
```

### `cademi integrations credentials secret-rotations` — Rotate a credential secret

Related: `cademi auth`, `cademi profiles`, `cademi account capabilities`

#### `cademi integrations credentials secret-rotations create <credential_id>`

Rotate a credential secret · `POST /api/v3/credentials/{credential_id}/secret-rotations` · permission `credentials.manage`

Generates a new secret for the credential. The credential ID and its policies do not change.

The previous secret remains valid for an overlap period, set by `overlap_hours` (1 to 168 hours, default 24), so integrations can switch to the new secret without downtime.

The new secret is returned only in this response and cannot be retrieved again. Store it securely. Retrying the request with the same `Idempotency-Key` does not return the secret again; it returns `409` with the `secret_not_replayable` error code.

**Arguments:**
- `credential_id` — Public ID of the credential, prefixed with `key_`. Example: `key_01J8Z3`

**Flag sets:** output, body, idempotency

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `overlap_hours` | integer, nullable |  |  |

Full schema: `cademi commands integrations credentials secret-rotations create --schema --json`

Legacy path: `cademi credentials secret-rotations create`

**Examples:**

```bash
# partial update
cademi integrations credentials secret-rotations create key_01J8Z3 -F overlap_hours=1 --json

# full body from a file
cademi integrations credentials secret-rotations create key_01J8Z3 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
