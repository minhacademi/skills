---
name: cademi-cli-automations
description: "Manage learning and sales automation journeys — `cademi automations` (18 commands)"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Automations Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

Diamond funnels coordinate a lead's journey through lessons and an offer.
They connect learning content, access deliveries and webhook triggers.

**Related:** `cademi content products`, `cademi sales deliveries`, `cademi integrations webhooks`

Live catalog for this file: `cademi commands automations --json` (offline, no credential needed).

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

### `cademi automations diamonds` — Manage Diamond funnels, memberships and stage triggers

Creating a Diamond also creates its product, three lessons, internal access
delivery and stage triggers. The internal delivery grants lesson access;
the expected purchase delivery determines conversion. Memberships track
each lead's stage, and triggers communicate transitions through webhooks.

Related: `cademi sales deliveries`, `cademi integrations webhooks`

#### `cademi automations diamonds create`

Create a Diamond · `POST /api/v3/automations/diamond` · permission `diamonds.create`, `products.create`, `deliveries.create`

Creates a Diamond automation from an existing showcase.

The operation also creates the Diamond's product, its three lessons, the internal delivery that grants access to those lessons, and one trigger for each of the eleven stages. Credentials scoped to the showcase can access all of these resources.

In addition to `diamonds.create`, this operation requires the `products.create` and `deliveries.create` permissions.

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string, nullable |  |  |
| `showcase_id` | string | yes |  |
| `type` | string |  | One of: `diamond` |

Full schema: `cademi commands automations diamonds create --schema --json`

Legacy path: `cademi diamonds create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi automations diamonds create -f showcase_id=<showcase_id> --json

# full body from a file
cademi automations diamonds create --data @body.json --json
```

#### `cademi automations diamonds delete <diamond_id>`

Delete a Diamond · `DELETE /api/v3/automations/diamond/{diamond_id}` · permission `diamonds.delete`

Moves the Diamond to the trash.

A Diamond that still has memberships cannot be deleted, and deletion cannot be forced. Such requests return the `state_conflict` error code.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output, idempotency, confirm

Legacy path: `cademi diamonds delete`

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi automations diamonds delete dmd_42 --yes
```

#### `cademi automations diamonds get <diamond_id>`

Retrieve a Diamond · `GET /api/v3/automations/diamond/{diamond_id}` · permission `diamonds.read`

Retrieves a Diamond with its settings and the IDs of its related product, showcase, delivery, and lessons.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the Diamond or one of its stages to avoid overwriting a newer version.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output

Legacy path: `cademi diamonds get`

**Examples:**

```bash
# get
cademi automations diamonds get dmd_42 --json
```

#### `cademi automations diamonds list`

List Diamonds · `GET /api/v3/automations/diamond` · permission `diamonds.read`

Returns the Diamonds accessible with the current credentials, paginated by cursor. Credentials scoped to specific showcases or products only see the Diamonds that belong to them.

The collection can be filtered by product, showcase, status, and deletion state.

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `--deleted` — When 'true', returns only items in the trash. Items in the trash are excluded by default.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--product-id <string>` — Only Diamonds of the product with this public ID.
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--showcase-id <string>` — Only Diamonds of the showcase with this public ID.
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--status <string>` — Only Diamonds with this status. (active, inactive)

Legacy path: `cademi diamonds list`

**Examples:**

```bash
# list: one page, machine-readable
cademi automations diamonds list --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi automations diamonds list --limit 200 --raw --json

# every page, projected
cademi automations diamonds list --all --jq '[.[] | {id}]'
```

#### `cademi automations diamonds update <diamond_id>`

Update a Diamond · `PATCH /api/v3/automations/diamond/{diamond_id}` · permission `diamonds.update`, `diamonds.activate (if field:status)`

Updates the name, status, or stage intervals of a Diamond. Stage intervals can be updated partially.

Updating `status` also requires the `diamonds.activate` permission.

Interval changes apply to future scheduling; communications already scheduled for leads are not affected.

Send the current `ETag` in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string |  |  |
| `settings` | object |  |  |
| `settings.steps_interval` | object |  |  |
| `status` | string |  | One of: `active`, `inactive` |

Full schema: `cademi commands automations diamonds update --schema --json`

Legacy path: `cademi diamonds update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi automations diamonds update dmd_42 -f name=<name> --if-match '"<etag>"' --json

# full body from a file
cademi automations diamonds update dmd_42 --data @body.json --json
```

### `cademi automations diamonds memberships` — Manage automations diamonds memberships

Related: `cademi sales deliveries`, `cademi integrations webhooks`

#### `cademi automations diamonds memberships create <diamond_id>`

Create a membership · `POST /api/v3/automations/diamond/{diamond_id}/memberships` · permission `diamonds.manage_memberships`, `enrollments.create`

Adds a user to the Diamond as a lead, starting at the `lead` stage. A new membership also grants the user access to the Diamond's internal delivery and schedules the communication for the `lead` stage.

If the user is already a member of this Diamond, the existing membership is returned and no communication is sent again. A user can belong to only one Diamond at a time.

In addition to `diamonds.manage_memberships`, this operation requires the `enrollments.create` permission.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `user_id` | string | yes |  |

Full schema: `cademi commands automations diamonds memberships create --schema --json`

Legacy path: `cademi diamonds memberships create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi automations diamonds memberships create dmd_42 -f user_id=<user_id> --json

# full body from a file
cademi automations diamonds memberships create dmd_42 --data @body.json --json
```

#### `cademi automations diamonds memberships get <diamond_id> <membership_id>`

Retrieve a membership · `GET /api/v3/automations/diamond/{diamond_id}/memberships/{membership_id}` · permission `diamonds.read`

Retrieves a lead in the Diamond, including its current stage, confirmed progress, and pending scheduled communication.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when moving the membership to another stage to avoid overwriting a newer version.

Credentials scoped to specific users can only access memberships of those users.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `membership_id` — Public ID of the membership, prefixed with `mbr_`. Example: `mbr_7`

**Flag sets:** output

Legacy path: `cademi diamonds memberships get`

**Examples:**

```bash
# get
cademi automations diamonds memberships get dmd_42 mbr_7 --json
```

#### `cademi automations diamonds memberships list <diamond_id>`

List memberships · `GET /api/v3/automations/diamond/{diamond_id}/memberships` · permission `diamonds.read`

Returns the leads in a Diamond, paginated by cursor. The collection can be filtered by user, stage, and last update time.

Credentials scoped to specific users only see the memberships of those users.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output-basic

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `--cursor <string>` — Cursor for the next page, taken from 'page.next_cursor' of the previous response.
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--limit <int64>` — Maximum number of items to return, from 1 to 200; minimum: 1; maximum: 200
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--sort <string>` — Sort order. A leading '-' sorts in descending order. (id, -id)
- `--stage-id <string>` — Only memberships currently in this stage.
- `--updated-after <string>` — Only items updated after this date and time (ISO 8601).
- `--user-id <string>` — Only memberships of the user with this public ID.

Legacy path: `cademi diamonds memberships list`

**Examples:**

```bash
# list: one page, machine-readable
cademi automations diamonds memberships list dmd_42 --limit 20 --json

# next page: pass page.next_cursor from a --raw response
cademi automations diamonds memberships list dmd_42 --limit 200 --raw --json

# every page, projected
cademi automations diamonds memberships list dmd_42 --all --jq '[.[] | {id}]'
```

#### `cademi automations diamonds memberships update <diamond_id> <membership_id>`

Update a membership stage · `PATCH /api/v3/automations/diamond/{diamond_id}/memberships/{membership_id}` · permission `diamonds.manage_memberships`

Moves a lead to a later stage of the Diamond.

Leads only move forward: a target stage at or before the current stage is rejected with the `state_conflict` error code, including a repeat of a transition that already happened. A successful transition sends the communication for the new stage once.

`reason` is required and is included in the `diamond_membership.stage_changed` event.

Send the membership's current `ETag` in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `membership_id` — Public ID of the membership, prefixed with `mbr_`. Example: `mbr_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | yes | Why the membership changes stage. It is sent as written in the `diamond_membership.stage_changed` event, which webhooks receive at every payload detail level, including `ids`. Do not include personal data. |
| `target_stage_id` | string | yes |  |

Full schema: `cademi commands automations diamonds memberships update --schema --json`

Legacy path: `cademi diamonds memberships update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi automations diamonds memberships update dmd_42 mbr_7 -f reason=<reason> -f target_stage_id=<target_stage_id> --json

# full body from a file
cademi automations diamonds memberships update dmd_42 mbr_7 --data @body.json --json
```

### `cademi automations diamonds memberships processing-attempts` — Manage automations diamonds memberships processing-attempts

Related: `cademi sales deliveries`, `cademi integrations webhooks`

#### `cademi automations diamonds memberships processing-attempts create <diamond_id> <membership_id>`

Create a processing attempt · `POST /api/v3/automations/diamond/{diamond_id}/memberships/{membership_id}/processing-attempts` · permission `diamonds.reprocess`

Requests that the most recent failed communication for the membership be sent again. Failed communications are not retried automatically.

The communication is queued for delivery. If the corresponding stage has already been confirmed, the message is not sent again, preventing duplicates.

A membership with no recorded failure is rejected with `409 state_conflict` and `details[].reason` = `no_failed_run`.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `membership_id` — Public ID of the membership, prefixed with `mbr_`. Example: `mbr_7`

**Flag sets:** output, idempotency, async

Legacy path: `cademi diamonds memberships processing-attempts create`

**Examples:**

```bash
# run
cademi automations diamonds memberships processing-attempts create dmd_42 mbr_7 --wait --json
```

#### `cademi automations diamonds memberships processing-attempts get <diamond_id> <membership_id> <attempt_id>`

Retrieve a processing attempt · `GET /api/v3/automations/diamond/{diamond_id}/memberships/{membership_id}/processing-attempts/{attempt_id}` · permission `diamonds.read`

Retrieves a processing attempt requested for the membership.

Only attempts that belong to the membership in the path can be retrieved; attempts of other memberships return `404 Not Found`.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `membership_id` — Public ID of the membership, prefixed with `mbr_`. Example: `mbr_7`
- `attempt_id` — Public ID of the attempt, prefixed with `pat_`. Example: `pat_9`

**Flag sets:** output

Legacy path: `cademi diamonds memberships processing-attempts get`

**Examples:**

```bash
# get
cademi automations diamonds memberships processing-attempts get dmd_42 mbr_7 pat_9 --json
```

#### `cademi automations diamonds memberships processing-attempts list <diamond_id> <membership_id>`

List processing attempts · `GET /api/v3/automations/diamond/{diamond_id}/memberships/{membership_id}/processing-attempts` · permission `diamonds.read`

Returns the processing attempts requested for the membership. All attempts are returned in a single response, without pagination.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `membership_id` — Public ID of the membership, prefixed with `mbr_`. Example: `mbr_7`

**Flag sets:** output

Legacy path: `cademi diamonds memberships processing-attempts list`

**Examples:**

```bash
# get
cademi automations diamonds memberships processing-attempts list dmd_42 mbr_7 --json
```

### `cademi automations diamonds stages` — Manage automations diamonds stages

Related: `cademi sales deliveries`, `cademi integrations webhooks`

#### `cademi automations diamonds stages list <diamond_id>`

List stages · `GET /api/v3/automations/diamond/{diamond_id}/stages` · permission `diamonds.read`

Returns all eleven stages of the Diamond, in order. The set of stages is fixed: stages cannot be created or deleted.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output

Legacy path: `cademi diamonds stages list`

**Examples:**

```bash
# get
cademi automations diamonds stages list dmd_42 --json
```

#### `cademi automations diamonds stages update <diamond_id> <stage_id>`

Update a stage · `PATCH /api/v3/automations/diamond/{diamond_id}/stages/{stage_id}` · permission `diamonds.update`

Updates the waiting interval of a time-based stage. Only time-based stages accept an interval, and only these values of `interval_minutes` are accepted: `15`, `30`, `60`, `120`, `180`, `360`, `720`, `1440`, and `2880`. The same values apply to `settings.steps_interval`.

The change applies to future scheduling; communications already scheduled for leads are not affected.

Stages share the Diamond's revision: send the Diamond's current `ETag` in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `stage_id` — Public ID of the stage, prefixed with `class_`. Example: `class_1_not_initiated`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `interval_minutes` | integer | yes |  |

Full schema: `cademi commands automations diamonds stages update --schema --json`

Legacy path: `cademi diamonds stages update`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi automations diamonds stages update dmd_42 class_1_not_initiated -F interval_minutes=1 --json

# full body from a file
cademi automations diamonds stages update dmd_42 class_1_not_initiated --data @body.json --json
```

### `cademi automations diamonds triggers` — Manage automations diamonds triggers

Related: `cademi sales deliveries`, `cademi integrations webhooks`

#### `cademi automations diamonds triggers create <diamond_id>`

Ensure a stage trigger · `POST /api/v3/automations/diamond/{diamond_id}/triggers` · permission `diamonds.manage_triggers`

Ensures that the trigger for a given stage exists. Every stage already has a trigger when the Diamond is created, so this operation normally returns the existing trigger; it creates a new one only if the stage's trigger is missing.

Changes to triggers affect future communications with leads.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output, body, idempotency

**Body (required):**

| Field | Type | Required | Description |
|---|---|---|---|
| `stage_id` | string | yes |  |

Full schema: `cademi commands automations diamonds triggers create --schema --json`

Legacy path: `cademi diamonds triggers create`

**Examples:**

```bash
# create: required fields with -f (strings) and -F (typed)
cademi automations diamonds triggers create dmd_42 -f stage_id=<stage_id> --json

# full body from a file
cademi automations diamonds triggers create dmd_42 --data @body.json --json
```

#### `cademi automations diamonds triggers get <diamond_id> <trigger_id>`

Retrieve a trigger · `GET /api/v3/automations/diamond/{diamond_id}/triggers/{trigger_id}` · permission `diamonds.read`

Retrieves a trigger of the Diamond. The destination is masked in the response.

The response includes an `ETag` representing the current revision. Send this value in the `If-Match` header when updating the trigger to avoid overwriting a newer version.

Only triggers that belong to the Diamond in the path can be retrieved.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `trigger_id` — Public ID of the trigger, prefixed with `trg_`. Example: `trg_7`

**Flag sets:** output

Legacy path: `cademi diamonds triggers get`

**Examples:**

```bash
# get
cademi automations diamonds triggers get dmd_42 trg_7 --json
```

#### `cademi automations diamonds triggers list <diamond_id>`

List triggers · `GET /api/v3/automations/diamond/{diamond_id}/triggers` · permission `diamonds.read`

Returns the triggers of the Diamond, one per stage.

Destinations are masked: only the URL host or the last digits of the phone number are returned.

Returns all triggers in a single response while preserving the standard collection response format.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`

**Flag sets:** output

Legacy path: `cademi diamonds triggers list`

**Examples:**

```bash
# get
cademi automations diamonds triggers list dmd_42 --json
```

#### `cademi automations diamonds triggers update <diamond_id> <trigger_id>`

Update a trigger · `PATCH /api/v3/automations/diamond/{diamond_id}/triggers/{trigger_id}` · permission `diamonds.manage_triggers`

Configures a stage trigger: whether it is enabled, its channel, destination, and template. The destination field must match the selected channel.

HTTP destinations must resolve to a public host; private, loopback, and metadata addresses are rejected.

The change applies to future communications with leads; nothing is sent when the trigger is updated.

Send the trigger's current `ETag` in the `If-Match` header to avoid overwriting a newer version.

**Arguments:**
- `diamond_id` — Public ID of the diamond, prefixed with `dmd_`. Example: `dmd_42`
- `trigger_id` — Public ID of the trigger, prefixed with `trg_`. Example: `trg_7`

**Flag sets:** output, body, idempotency

**Flags:**
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `channel` | string |  | One of: `http`, `whatsapp`, `sms`, `email` |
| `destination` | object, nullable |  |  |
| `destination.email` | string |  |  |
| `destination.phone` | string |  |  |
| `destination.url` | string |  |  |
| `enabled` | boolean |  |  |
| `template` | string, nullable |  |  |

Full schema: `cademi commands automations diamonds triggers update --schema --json`

Legacy path: `cademi diamonds triggers update`

**Examples:**

```bash
# partial update guarded by the ETag from get -i
cademi automations diamonds triggers update dmd_42 trg_7 -f channel=http --if-match '"<etag>"' --json

# full body from a file
cademi automations diamonds triggers update dmd_42 trg_7 --data @body.json --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
