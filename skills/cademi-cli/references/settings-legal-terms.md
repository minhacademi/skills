---
name: cademi-cli-settings-legal-terms
description: "Manage legal terms presented to students — `cademi settings legal-terms` (6 commands)"
metadata:
  cademi-cli: "0.2.5"
  cademi-api: "3.12.0"
---

# Settings Legal Terms Commands

> cademi 0.2.5, API 3.12.0. The live catalog is always `cademi commands <prefix> --json`.

> Domain `cademi settings` — Configure platform behavior, appearance and shared definitions.

Define legal terms here. A student's recorded acceptances are available
through users term-acceptances.

**Related:** `cademi users term-acceptances`

**See also in this domain:** `references/settings-access.md`, `references/settings-definitions.md`, `references/settings-platform.md`, `references/settings-emails.md`, `references/settings-menus.md`, `references/settings-support.md`

Live catalog for this file: `cademi commands settings legal-terms --json` (offline, no credential needed).

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

## Commands

### `cademi settings legal-terms create`

Create a legal term · `POST /api/v3/legal-terms` · permission `legal_terms.create`

Configures the legal term for a scope. There is at most one legal term for the platform and one per product, and legal terms are not versioned.

The `product` scope requires `product_id`. If the scope already has a legal term, the request is rejected; use the update operation to change its text.

Legal terms of replicated accounts and replicated products are read-only.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | object | yes |  |
| `body.blocks` | array of object | yes |  |
| `product_id` | string, nullable |  |  |
| `scope` | string | yes | One of: `platform`, `product` |

Full schema: `cademi commands settings legal-terms create --schema --json`

Legacy path: `cademi legal-terms create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings legal-terms create -F 'body={...}' --data '{"body.blocks":[...]}' -f scope=platform --json

# full body from a file
cademi settings legal-terms create --data @body.json --json
```

### `cademi settings legal-terms get <legal_term_id>`

Retrieve a legal term · `GET /api/v3/legal-terms/{legal_term_id}` · permission `legal_terms.read`

Retrieves a legal term, including its body.

The legal term ID is derived from its scope: `trm_platform` for the platform term, or `trm_product_<n>` for a product term, where `<n>` is the numeric part of the product ID (for example, `trm_product_42` for product `prd_42`).

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the legal term to avoid overwriting a newer version.

**Arguments:**
- `legal_term_id` — Public ID of the legal term, prefixed with `trm_`. Example: `trm_platform`

**Flag sets:** output

Legacy path: `cademi legal-terms get`

**Examples:**

```bash
# get
cademi settings legal-terms get trm_platform --json
```

### `cademi settings legal-terms list`

List legal terms · `GET /api/v3/legal-terms` · permission `legal_terms.read`

Lists the configured legal terms: at most one for the platform and one per product. The collection can be filtered by scope and product.

The term body is not included in the collection; use the retrieve operation to obtain it. When the credentials are restricted to specific legal terms, only those terms are returned.

Returns all matching legal terms in a single response while preserving the standard collection response format.

**Flag sets:** output

**Flags:**
- `--product-id <string>` — Only the term of the product with this public ID.
- `--scope <string>` — Only the platform term or only product terms. (platform, product)

Legacy path: `cademi legal-terms list`

**Examples:**

```bash
# get
cademi settings legal-terms list --json
```

### `cademi settings legal-terms update <legal_term_id>`

Update a legal term · `PATCH /api/v3/legal-terms/{legal_term_id}` · permission `legal_terms.update`

Replaces the body of a legal term. Setting `body` to `null` removes the legal term; subsequent requests for it return `404`.

Changing the text does not invalidate acceptances already recorded, and users are not asked to accept the term again.

Send the `ETag` returned by the retrieve operation in the `If-Match` header to avoid overwriting a newer version. Legal terms of replicated accounts and replicated products are read-only.

**Arguments:**
- `legal_term_id` — Public ID of the legal term, prefixed with `trm_`. Example: `trm_platform`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `body` | object, nullable | yes |  |
| `body.blocks` | array of object | yes |  |

Full schema: `cademi commands settings legal-terms update --schema --json`

Legacy path: `cademi legal-terms update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi settings legal-terms update trm_platform -F 'body={...}' --data '{"body.blocks":[...]}' --json

# full body from a file
cademi settings legal-terms update trm_platform --data @body.json --json
```

### `cademi settings legal-terms acceptances` — Manage settings legal-terms acceptances

#### `cademi settings legal-terms acceptances get <legal_term_id> <term_acceptance_id>`

Retrieve a legal term acceptance · `GET /api/v3/legal-terms/{legal_term_id}/acceptances/{term_acceptance_id}` · permission `legal_terms.read`

Retrieves an acceptance of a legal term by its public ID. This is the canonical location of every acceptance, including those returned in a user's acceptance history.

The `proof` object, containing the IP address and user agent recorded at acceptance, is included only when the credentials have the `legal_terms.read_proof` permission. Otherwise, it is omitted from the response.

**Arguments:**
- `legal_term_id` — Public ID of the legal term, prefixed with `trm_`. Example: `trm_platform`
- `term_acceptance_id` — Public ID of the term acceptance, prefixed with `tac_`. Example: `tac_9`

**Flag sets:** output

Legacy path: `cademi legal-terms acceptances get`

**Examples:**

```bash
# get
cademi settings legal-terms acceptances get trm_platform tac_9 --json
```

#### `cademi settings legal-terms acceptances list <legal_term_id>`

List legal term acceptances · `GET /api/v3/legal-terms/{legal_term_id}/acceptances` · permission `legal_terms.read`

Lists the acceptances recorded for a legal term. Acceptances are recorded when users accept the term and cannot be created through the API.

The collection can be filtered by user and by acceptance date. When the credentials are restricted to specific users, only acceptances by those users are returned.

The `proof` object is included only when the credentials have the `legal_terms.read_proof` permission.

**Arguments:**
- `legal_term_id` — Public ID of the legal term, prefixed with `trm_`. Example: `trm_platform`

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
- `--user-id <string>` — Only acceptances by the user with this public ID.

Legacy path: `cademi legal-terms acceptances list`

**Examples:**

```bash
# list: one page, machine-readable
cademi settings legal-terms acceptances list trm_platform --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi settings legal-terms acceptances list trm_platform --limit 200 --raw --json

# every page, projected
cademi settings legal-terms acceptances list trm_platform --all --jq '[.[] | {id}]'
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
