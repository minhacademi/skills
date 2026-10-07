---
name: cademi-cli-sandbox
description: "Inspect sandbox data and run test scenarios — `cademi sandbox` (7 commands)"
metadata:
  cademi-cli: "0.2.7"
  cademi-api: "3.12.1"
---

# Sandbox Commands

> cademi 0.2.7, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

Sandbox workflows reset test data and run predefined scenarios. reset and
run require sandbox credentials. Inspect auth status to check your environment.

**Related:** `cademi auth`, `cademi profiles`

Live catalog for this file: `cademi commands sandbox --json` (offline, no credential needed).

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

### `cademi sandbox get`

Retrieve the sandbox · `GET /api/v3/sandbox` · permission `sandbox.read`

Describes the sandbox associated with the current credentials.

With production credentials, the response describes the linked sandbox. If no sandbox has been provisioned, the request still succeeds and `status` is `absent`. With sandbox credentials, the response describes the sandbox itself.

**Flag sets:** output

**Examples:**

```bash
# get
cademi sandbox get --json
```

### `cademi sandbox reset`

Erase the sandbox data and seed it again

Delete the sandbox data (including uploaded files) and seed the fixed test data again.
Credentials, administrators, limits and the audit trail are kept. Production is never affected.

**Flag sets:** output-basic, confirm

**Flags:**
- `--no-wait` — Do not wait for the reset to finish

**Examples:**

```bash
# delete: confirms unless --yes (required without a terminal)
cademi sandbox reset --yes
```

### `cademi sandbox run <scenario_id>`

Run a test scenario and show what it created

Run a sandbox test scenario. It creates records marked as simulated and emits the
same public events as production, which reach webhooks and `cademi listen`.
List the scenarios and their parameters with `cademi sandbox test-scenarios list`.

**Arguments:**
- `scenario_id`

**Flag sets:** output-basic

**Flags:**
- `--no-wait` — Do not wait for the scenario to finish
- `--param <stringArray>` — Scenario parameter: key=value (typed like -F)

**Examples:**

```bash
cademi sandbox run scn_01J8Z3...
cademi sandbox run scn_01J8Z3... --param users=3 --param product_name="Course A"
```

### `cademi sandbox resets` — Reset the sandbox

#### `cademi sandbox resets create`

Reset the sandbox · `POST /api/v3/sandbox/resets` · permission `sandbox.manage`

Deletes the sandbox data, including uploaded files, and seeds the default test data again. Credentials, administrators, and audit entries are preserved, and production data is never affected.

Only sandbox credentials can request a reset. Production credentials receive `403 Forbidden` with the `sandbox_only` error code. `POST /sandbox/test-scenarios/{scenario_id}/runs` refuses the same credential with `409 sandbox_required`: both codes mean the request must be made with a sandbox credential, and a client can handle them the same way. Only one reset can be queued or running at a time.

The reset is processed asynchronously. The `Location` header points to the operation that tracks its progress. The `Idempotency-Key` header is required; retrying with the same key returns the same operation.

**Flag sets:** output, body, idempotency, async

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string, nullable |  |  |

Full schema: `cademi commands sandbox resets create --schema --json`

**Examples:**

```bash
# partial update
cademi sandbox resets create -f reason=<reason> --wait --json

# full body from a file
cademi sandbox resets create --data @body.json --wait --json
```

### `cademi sandbox test-scenarios` — Manage sandbox test-scenarios

#### `cademi sandbox test-scenarios get <scenario_id>`

Retrieve a test scenario · `GET /api/v3/sandbox/test-scenarios/{scenario_id}` · permission `sandbox.read`

Retrieves a test scenario from the sandbox catalog, including the parameters it accepts and the effects it produces. Scenario IDs use the `scn_` prefix.

**Arguments:**
- `scenario_id` — Public ID of the scenario, prefixed with `scn_`. Example: `scn_sandbox`

**Flag sets:** output

**Examples:**

```bash
# get
cademi sandbox test-scenarios get scn_sandbox --json
```

#### `cademi sandbox test-scenarios list`

List test scenarios · `GET /api/v3/sandbox/test-scenarios` · permission `sandbox.read`

Returns the complete catalog of sandbox test scenarios in a single response while preserving the standard collection response format.

The catalog is the same for production and sandbox credentials.

**Flag sets:** output

**Examples:**

```bash
# get
cademi sandbox test-scenarios list --json
```

### `cademi sandbox test-scenarios runs` — Run a test scenario

#### `cademi sandbox test-scenarios runs create <scenario_id>`

Run a test scenario · `POST /api/v3/sandbox/test-scenarios/{scenario_id}/runs` · permission `sandbox.manage`

Runs a test scenario in the sandbox. Records created by the scenario have `meta.simulated` set to `true`, and the corresponding events are delivered to webhooks and event streams as they would be in production. Scenario parameters can be supplied in `params`.

Available only with sandbox credentials. Production credentials receive `409 Conflict` with the `sandbox_required` error code. `POST /sandbox/resets` refuses the same credential with `403 sandbox_only`: both codes mean the request must be made with a sandbox credential, and a client can handle them the same way.

The run is processed asynchronously. The `Location` header points to the operation that tracks its progress. The `Idempotency-Key` header is required; retrying with the same key returns the same operation without running the scenario again.

Resetting the sandbox deletes the data created by scenario runs.

**Arguments:**
- `scenario_id` — Public ID of the scenario, prefixed with `scn_`. Example: `scn_sandbox`

**Flag sets:** output, body, idempotency, async

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `params` | object |  |  |

Full schema: `cademi commands sandbox test-scenarios runs create --schema --json`

**Examples:**

```bash
# partial update
cademi sandbox test-scenarios runs create scn_sandbox -F 'params={...}' --wait --json

# full body from a file
cademi sandbox test-scenarios runs create scn_sandbox --data @body.json --wait --json
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
