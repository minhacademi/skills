---
name: cademi-cli-support
description: "Handle student comments, questions and support tickets — `cademi support` (5 commands)"
metadata:
  cademi-cli: "0.2.7"
  cademi-api: "3.12.1"
---

# Support Commands

> cademi 0.2.7, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

Comments, questions and tickets are student communication channels.
Departments organize support and FAQs provide reusable answers.

**Related:** `cademi reports support`

**Related settings:**
- `cademi settings support` — Controls availability of the student support channel and how administrators open conversations. (`cademi settings support get|update`)
- `cademi settings support comments` — Controls lesson comments, publication moderation, spam limits and the administrator inbox. (`cademi settings support comments get|update`)
- `cademi settings support questions` — Controls lesson questions, publication moderation and the administrator inbox. (`cademi settings support questions get|update`)
- `cademi settings support faq` — Controls whether the FAQ page is available and the title and guidance shown above its answers. (`cademi settings support faq get|update`)

**See also in this domain:** `references/support-comments.md`, `references/support-faqs.md`, `references/support-questions.md`, `references/support-tickets.md`

**Group examples:**

```bash
cademi support questions list
cademi support tickets list
```

Live catalog for this file: `cademi commands support --json` (offline, no credential needed).

## Shared flag sets

Each command lists the sets it accepts. The flags of a set are:

**output**
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document)
- `--jq <string>` — Filter data (or --raw envelope); strings print unquoted, other values as JSON; overrides --output
- `--json` — Print JSON (same as --output json)
- `-o, --output <string>` — Output format: json, table or yaml (default: table for lists on a terminal, JSON otherwise)
- `--raw` — Print the full response envelope instead of data

**body**
- `-d, --data <string>` — JSON body: a literal, @file or @- for stdin
- `-f, --field <stringArray>` — Add a string field: key=value (nested: a.b=v, arrays: tags[]=v)
- `-F, --typed-field <stringArray>` — Add a typed field: key=true\|false\|null\|number\|JSON, or key=@file to read text; applied after all -f fields

**idempotency**
- `--idempotency-key <string>` — Reuse the key returned with a failed request to repeat it (default: automatic UUIDv7)

**confirm**
- `-y, --yes` — Do not ask for confirmation

## Commands

### `cademi support departments` — Organize support tickets into departments

Manage the departments used by the support ticket workflow.

Related: `cademi support tickets`

Related settings:
- `cademi settings support` — Controls availability of the student support channel and how administrators open conversations. (`cademi settings support get|update`)

#### `cademi support departments create`

Create a department · `POST /api/v3/support/departments` · permission `departments.create`

Creates a support department with the given name. `admin_id` assigns the responsible administrator.

The response includes a `Location` header pointing to the new department.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `admin_id` | string, nullable |  |  |
| `name` | string | yes |  |

Full schema: `cademi commands support departments create --schema --json`

Legacy path: `cademi departments create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi support departments create -f name=<name> --json

# full body from a file
cademi support departments create --data @body.json --json
```

#### `cademi support departments delete <department_id>`

Delete a department · `DELETE /api/v3/support/departments/{department_id}` · permission `departments.delete`

Deletes a support department.

A department that still has tickets assigned to it, including deleted tickets, cannot be deleted unless `unlink_tickets=true` is sent. Without that flag the request is rejected with the `state_conflict` error code. With the flag, tickets are kept and unlinked, matching the dashboard, including deleted tickets, which the API does not list. To keep tickets in a department, move them first with `PATCH /support/tickets/{ticket_id}` (`department_id`).

**Arguments:**
- `department_id` — Public ID of the department, prefixed with `dep_`. Example: `dep_3`

**Flag sets:** output, idempotency, confirm

**Flags:**
- `--unlink-tickets` — When true, unassign tickets (including those in the trash) and delete the department.

Legacy path: `cademi departments delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi support departments delete dep_3 --yes
```

#### `cademi support departments get <department_id>`

Retrieve a department · `GET /api/v3/support/departments/{department_id}` · permission `departments.read`

Retrieves a support department by its public ID.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the department to avoid overwriting a newer version.

**Arguments:**
- `department_id` — Public ID of the department, prefixed with `dep_`. Example: `dep_3`

**Flag sets:** output

Legacy path: `cademi departments get`

**Examples:**

```bash
# get
cademi support departments get dep_3 --json
```

#### `cademi support departments list`

List departments · `GET /api/v3/support/departments` · permission `departments.read`

Returns all support departments of the current account in a single response. Each department includes the number of tickets assigned to it in `tickets_count`.

**Flag sets:** output

Legacy path: `cademi departments list`

**Examples:**

```bash
# get
cademi support departments list --json
```

#### `cademi support departments update <department_id>`

Update a department · `PATCH /api/v3/support/departments/{department_id}` · permission `departments.update`

Updates a support department. Omitted fields are left unchanged. Send `admin_id` as `null` to clear the assigned administrator.

The `If-Match` header is optional. When supplied, it must contain the `ETag` returned by the retrieve operation; if the department has changed since then, the update is rejected.

**Arguments:**
- `department_id` — Public ID of the department, prefixed with `dep_`. Example: `dep_3`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `admin_id` | string, nullable |  |  |
| `name` | string |  |  |

Full schema: `cademi commands support departments update --schema --json`

Legacy path: `cademi departments update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi support departments update dep_3 -f admin_id=<admin_id> --if-match '"<etag>"' --json

# full body from a file
cademi support departments update dep_3 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
