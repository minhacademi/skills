---
name: cademi-cli
description: Guide for using the cademi CLI to operate a Cademí platform (API v3) from the terminal - students, enrollments, products, modules, lessons, showcases, deliveries and sales, webhooks and events, settings, support, reports, files, sandbox scenarios and config-as-code. Use when the user mentions Cademí, `cademi` commands, Cademí IDs (usr_, prd_, dlv_, enr_, op_...), CADEMI_API_KEY, or wants to inspect or automate a Cademí account from a terminal, a script or CI, and when debugging cademi exit codes or JSON errors.
license: Apache-2.0
compatibility: Requires the cademi CLI 0.2 or newer (curl -fsSL https://cli.cademi.dev/install.sh | bash) and a Cademí credential (CADEMI_API_KEY or `cademi auth login`).
metadata:
  author: minhacademi
  version: "0.1.3"
  cademi-cli: "0.2.2"
  cademi-api: "3.10.1"
  docs: https://cademi.dev/cli
---

# Cademí CLI Usage Guide

Help users operate a Cademí platform from the command line with the `cademi` CLI. Every API v3 operation is a command, grouped by domain (`cademi content products list`), next to built-in commands for authentication, raw requests, events, files and configuration as code.

> **Core rule for agents: never guess a command path, a flag or a body field.** If `cademi` is not installed, install it first ([Installing the CLI](#installing-the-cli)), then let the CLI describe itself. The CLI ships its own catalog. Read the reference files in `references/` or run `cademi commands <prefix> --json` (offline, no credential needed). Unknown fields fail locally with exit code 2 before anything reaches the API, so a guess costs a round trip and teaches you nothing. Do **not** run `cademi auth login` pre-emptively: a missing credential exits 3 with a clear message, and the fix is the user's (set `CADEMI_API_KEY` or log in), never a fabricated key.

## Agent Guidance

### Key Principles

- **Command shape**: `cademi <domain> <group…> <verb> <ids…>`: path parameters are positional (`cademi content products modules get prd_42 mod_7`), query parameters are kebab-case flags (`--status published`, `--tag-id tag_1,tag_2`), bodies come from `-d/--data`, `-f/--field` and `-F/--typed-field`.
- **Prefer generated resource commands over `cademi api`**: they validate input against the contract, type the flags and document the body. Use `cademi api <METHOD> <path>` only for operations newer than the CLI or when mirroring a raw HTTP call.
- **Use `--json` (or `--jq`) whenever you parse output**: lists print as tables on a terminal and as JSON when piped; `--json` removes the ambiguity. Standard output carries only data; status, progress and errors go to standard error.
- **Read the exit code, not the text**: codes are stable (table below); messages follow `CADEMI_LANG` and change between releases.
- **Check the body before writing**: `cademi commands <cmd> --schema --json` returns the complete request schema (types, enums, nested fields) offline. Reference files list the fields; `--schema` has the details.
- **Writes are safe to retry**: every POST/PATCH/PUT/DELETE carries an automatic `Idempotency-Key`, and the CLI retries network errors, 429 and 5xx on its own. A failed write prints the key it used so you can replay it.
- **Canonical paths only**: `cademi content products list`, not the hidden legacy `cademi products list`. Legacy paths still run but do not appear in `--help` or `cademi commands`, and may be removed.
- **IDs are prefixed**: `usr_` user, `prd_` product, `mod_` module, `les_` lesson, `shw_` showcase, `dlv_` delivery, `enr_` enrollment, `exm_` exam, `cfd_` custom field, `tag_` tag, `tkt_` ticket, `whk_` webhook, `whd_` webhook delivery, `str_` event stream, `evt_` event, `op_` operation, `file_` file, `scn_` sandbox scenario, `key_` credential. The catalog shows the prefix of every path argument (`arguments[].schema.example`). A 404 on a well-formed call usually means a wrong prefix or an ID outside the credential's scope.
- **Access is delivery + enrollment**: a delivery (`cademi sales deliveries`) defines what a purchase or a manual grant gives; a user gets content through an enrollment in a delivery (`cademi users enrollments create <user_id> -f delivery_id=dlv_…`). Publishing content is a separate step (`publish-all`).
- **Sandbox for experiments**: credentials starting with `ck_test_` hit the sandbox; `cademi sandbox run <scenario_id>` creates simulated data and real events. Production profiles are highlighted and destructive prompts start with `[production]`.
- **Docs when the catalog cannot answer**: https://cademi.dev/cli/*.md (index at https://cademi.dev/llms.txt) explain concepts; the catalog and `cademi guide <topic> --help` explain the commands.

### Discovery Flow

The CLI describes itself. Narrow one level at a time instead of dumping everything:

```bash
# 1. Domains and core groups, compact (one call, ~15 lines)
cademi commands --groups --brief --json

# 2. Groups inside one domain
cademi commands content --groups --brief --json

# 3. Every command in one area, with arguments, flags, permissions and body fields
cademi commands content products modules --json

# 4. The complete request body of one command, with nested schemas and enums
cademi commands users update --schema --json --jq '.commands[0].request_body'

# Built-in commands and the five automation guides are plain text
cademi listen --help
cademi guide output --help
```

The files in `references/` hold the same catalog, rendered as Markdown and grouped by domain (see [Command Reference](#command-reference)). Open the file for the area you need instead of running step 3; run step 4 before any write with more than a couple of fields.

### Design Principles

The `cademi` CLI follows conventions from `gh` and `stripe`, so that knowledge transfers directly:

- **`<noun> <verb>` commands** with `list`, `get`, `create`, `update`, `delete` and a few domain verbs (`publish-all`, `order update`, `copies create`, `replays create`).
- **`gh api`-style input**: `-f key=value` for strings, `-F key=value` for typed values (`true`, `false`, `null`, numbers, JSON, `@file`), `-d '{…}'`, `-d @file.json` or `-d @-` for a full body. Nested fields use dots (`criteria.percent=80`) and `key[]=value` appends to arrays.
- **Built-in `--jq`** on every command (no `jq` binary needed), `-o json|table|yaml`, `--raw` for the full envelope and `-i` for status and headers, as in `gh`.
- **`cademi api` mimics `curl`**: `cademi api [METHOD] <path>` relative to `/api/v3`, with `-H` for headers and the same `-f/-F/-d` input.

### Authentication and Profiles

| Mode | How | Use it for |
|---|---|---|
| Autonomous | `CADEMI_API_KEY=ck_…` in the environment (overrides every profile, no keychain) or `cademi auth login --api-key-only` | Scripts, CI, containers, agents |
| Human | `cademi auth login [--platform acme.cademi.com.br]`: browser OAuth with 2FA as an administrator, then paste a credential secret | People at a terminal; the audit trail records the administrator |
| Delegated | `CADEMI_ACCESS_TOKEN` issued by the Cademí MCP | Calls made on behalf of an MCP session |

- Precedence: `CADEMI_ACCESS_TOKEN`, then `CADEMI_API_KEY`, then the profile from `-p/--profile`, `CADEMI_PROFILE`, or the current one.
- Non-interactive login with a profile: `echo "$KEY" | cademi auth login --api-key-only --with-key --profile ci`.
- `cademi auth status --json` shows the profile, environment (`production` or `sandbox`), credential and permissions count; `cademi integrations credentials current-policies` lists the effective permissions. `cademi profiles list` and `cademi profiles use <name>` switch accounts.
- A machine without a keychain (container, headless Linux) must use `CADEMI_API_KEY`.
- `cademi doctor` checks config, keychain, network, clock, credential and API release compatibility in one run.

### Input: Bodies and Queries

- `-f name="Ana Souza"` sends a string, always. Booleans, numbers, null and JSON need `-F`: `-F send_credentials=false`, `-F 'settings={"block_comments":true}'`.
- Merge order is `--data` first, then every `-f`, then every `-F`. Repeated scalar keys overwrite; `tags[]=tag_1 -f tags[]=tag_2` appends. Arrays of objects need `--data`.
- Quote anything with brackets or JSON so the shell does not expand it.
- The CLI validates query parameters and body fields against the API release it was built for. An unknown field exits 2 and lists the accepted names; a field newer than the CLI needs `cademi api … --skip-validation` or `cademi update`.
- Missing required body: exit 2 with "use -f, -F or --data".

### Output and Streams

- Commands print the response `data`; `--raw` prints the whole envelope, including `page.next_cursor`.
- `--jq '<expr>'` filters the data (or the envelope with `--raw`); strings print unquoted, other values as JSON, one result per line. It runs once after `--all` collects every page.
- `-i/--include` prepends HTTP status and headers (`ETag`, `X-Request-Id`, `X-Cademi-Release`); the output is then not a JSON document.
- `--json` cannot be combined with `-o table` or `-o yaml`; `--all` cannot be combined with `--raw` or `-i` (exit 2 before any request).
- Progress of `--wait`, warnings such as `Idempotent replay`, and `Ready!` from `cademi listen` are on standard error.

### Exit Codes

| Code | Meaning | Agent action |
|---|---|---|
| 0 | Success, including an operation that ended `partially_succeeded` | Proceed; with `--wait`, check `status` and `cademi operations items list <op_id>` for item failures |
| 1 | Generic failure: operation `failed`/`canceled`, wait timeout, `doctor` check failed, denied login, network error after retries | Read stderr; a write may still have been applied, replay with its idempotency key rather than a new one |
| 2 | Usage or local validation: unknown command/flag/field, missing body, confirmation needed without a terminal, bad env var | Fix the invocation from the message (it lists accepted names); add `--yes` only if the user asked for a non-interactive run |
| 3 | Authentication: no profile, keychain secret missing, 401, session cannot be renewed | Stop and ask the user to set `CADEMI_API_KEY` or run `cademi auth login`; never invent a key |
| 4 | Permission: 403 | Report the permission named in the command's catalog entry (`permissions`); the credential's policy needs it |
| 5 | Not found: 404 | Check the ID prefix and that the resource is in scope of the credential |
| 6 | Rejected: other 4xx (409 `already_exists`/`state_conflict`, 412 `revision_mismatch`, 422 `validation_failed`) | Branch on `error.code`: re-read and retry on 412, fix `details[]` on 422, reuse the existing resource on `already_exists` |
| 7 | Rate limited: 429 after retries, or a wait longer than `CADEMI_MAX_RETRY_WAIT` | Back off; the CLI already retried up to 3 times |
| 8 | Server error: 5xx after retries | Retry later; include `request_id` if reporting |
| 130 | Canceled: Ctrl-C or a declined confirmation | Nothing was confirmed; ask before retrying |

### Errors as JSON

With `--json`, `-o json` or `--jq`, errors are one JSON document on **stderr**, in the API's own envelope:

```json
{"error":{"code":"validation_failed","message":"…","request_id":"req_…","details":[{"field":"email","code":"invalid","message":"…"}]}}
{"error":{"code":"cli_error","message":"no profile configured: run `cademi auth login` or set CADEMI_API_KEY"}}
{"error":{"code":"internal_error","message":"…","request_id":"…"},"request":{"method":"POST","path":"/api/v3/users","idempotency_key":"0192f3a1-…"}}
```

- `cli_error` is the CLI itself (flags, arguments, local validation); everything else comes from the API.
- Branch on `error.code`, never on `message`.
- `request_id` is safe to share and is what Cademí support needs. Secrets and personal data are not.
- `request.idempotency_key` appears when a write was sent and failed: replay it with `--idempotency-key <key>` and the same body.
- Warnings can precede the error on stderr, so the stream is not always a single JSON document.

### Writes: Idempotency, Confirmation, Conditional Updates, Async

- **Automatic idempotency**: each write gets a new UUIDv7 key; internal retries reuse it. To make a whole job rerunnable, choose the key yourself: `--idempotency-key "run-${CI_RUN_ID}-create-ana"`. Same key + same body within 48 hours returns the stored result (`Idempotent replay` warning). Never reuse a key with a different body.
- **Deletes confirm**: `delete` commands, `cademi sandbox reset` and `cademi config apply` ask `y/yes`; without a terminal they exit 2 unless you pass `--yes`/`-y`. Production profiles show `[production]` in the prompt.
- **Conditional updates**: `get … -i` exposes the `ETag`; `update … --if-match '<etag>'` fails with 412 (exit 6) if someone changed the resource first. Reordering (`… order update`) is documented in `cademi guide reordering --help`.
- **Asynchronous operations**: a `202` returns an operation (`op_…`). Add `--wait [--wait-timeout 10m]` to block, or `cademi operations wait <op_id> --timeout 2m --json` later. `failed`/`canceled` exit 1; a timeout exits 1 without canceling the server-side operation; `partially_succeeded` exits 0 with a warning.

### Pagination

- `--limit` is the page size (1–200 on most collections), not a total.
- Bounded walk: `… --limit 200 --raw --json`, read `page.next_cursor`, pass it to `--cursor` with the same filters and sort; stop when it is empty or null. Cursors are opaque.
- `--all` follows every page into one array in memory and prints nothing if any page fails. Pair it with `--jq` to shrink the result: `--all --jq '[.[] | {id, name}]'`.

### Safety Rules

- Confirm with the user before `delete`, `publish-all`, `order update`, `secret-rotations create`, `cademi sandbox reset`, `cademi config apply`, and any write against a `production` profile (`cademi auth status` shows the environment).
- Prefer a sandbox credential (`ck_test_`) or `--profile sandbox` for experiments; the sandbox never touches production.
- Never print, log or commit `ck_live_`/`ck_test_` secrets, OAuth tokens, `whsec_` secrets, or the `Authorization`/`X-API-Key` headers. `--debug` never logs them either.
- Never put personal data (names, emails, documents) or secrets into bug reports; the `request_id` is enough.
- Do not disable validation (`--skip-validation`) to make a call pass; fix the field or update the CLI.
- Do not run `cademi update` or an interactive `cademi auth login` from inside an automated loop.

### Context Window Tips

- Use `--brief` on `cademi commands` and `--groups` to list groups before commands.
- Project with `--jq`: `cademi users list --limit 20 --jq '[.[] | {id, email}]'` instead of the full objects.
- Ask for one command's schema: `cademi commands <cmd> --schema --json --jq '.commands[0].request_body'`.
- Read one reference file instead of `cademi commands --json` (600 KB).
- Keep `--limit` small while exploring; use `--all` only when the task needs every item.

### Workflow Patterns

#### Check who you are

```bash
# Profile, environment (production/sandbox), credential and API release
cademi auth status --json
# Effective permissions of the credential
cademi integrations credentials current-policies --json
# One-shot diagnosis when something fails
cademi doctor
```

#### Create a student and grant access

```bash
# 1. Does the user exist?
cademi users list --q ana@example.com --jq '[.[] | {id, email}]'

# 2. Create (strings with -f, typed values with -F)
cademi users create -f name="Ana Souza" -f email=ana@example.com -F send_credentials=false --jq .id

# 3. Pick the delivery that grants the content
cademi sales deliveries list --jq '[.[] | {id, name}]'

# 4. Enroll the user in the delivery
cademi users enrollments create usr_42 -f delivery_id=dlv_7 --json

# 5. Verify the effective access to a product
cademi users products access get usr_42 prd_12 --json
```

#### Publish a product and wait for the operation

```bash
cademi content products publish-all prd_12 --wait --wait-timeout 10m --json
# If the wait timed out, resume (the operation keeps running server-side)
cademi operations wait op_01J8Z3ZQ4H8K2M0T1S9P7YQF5C --timeout 5m --json
# Item-level outcome of a partially_succeeded operation
cademi operations items list op_01J8Z3ZQ4H8K2M0T1S9P7YQF5C --all --json
```

#### Build content: product → module → lesson

```bash
cademi content products create -f name="Course A" -f format=course -f showcase_id=shw_01J8Z1 --jq .id
cademi content products modules create prd_12 -f name="Module 1" --jq .id
cademi content products lessons create prd_12 -f name="Lesson 1" -f module_id=mod_7 --json
# Check the body of any of these before sending extra fields
cademi commands content products lessons create --schema --json --jq '.commands[0].request_body'
```

#### Update without clobbering a concurrent change

```bash
cademi content products get prd_12 -i --json          # read the ETag header
cademi content products update prd_12 -f name="New name" --if-match 'W/"product:prd_12:…"' --json
# exit 6 with error.code revision_mismatch → read again, rebuild the change, retry
```

#### Paginate

```bash
# Everything, projected (buffers all pages)
cademi users list --tag-id tag_7 --all --jq '[.[] | {id, email}]'
# Bounded: one page at a time
cademi users list --limit 200 --raw --json              # take .page.next_cursor
cademi users list --limit 200 --cursor '<next_cursor>' --raw --json
```

#### Replay a failed write

```bash
cademi users create --data @new-user.json --json
# stderr: {"error":{…},"request":{"method":"POST","path":"/api/v3/users","idempotency_key":"0192…"}}
cademi users create --data @new-user.json --json --idempotency-key '0192…'
```

#### Test a webhook receiver locally

```bash
# Terminal 1: forward signed events to the app under test (sandbox profile)
cademi --profile sandbox listen --print-secret            # whsec_local_… for signature checks
cademi --profile sandbox listen --forward-to localhost:3000/webhooks/cademi --events user.created,enrollment.created

# Terminal 2: produce events
cademi --profile sandbox sandbox test-scenarios list --json
cademi --profile sandbox sandbox run scn_01J8Z3 --param users=3

# Machine-readable stream
cademi listen --json | jq -c 'select(.type == "user.created")'
```

#### Configuration as code

```bash
cademi config validate catalog.yaml                  # exit 6 when invalid
cademi config plan catalog.yaml --out plan.json      # +create ~update -delete =unchanged !skip
cademi config apply --plan plan.json catalog.yaml --yes   # plan is valid 24h, same credential
```

#### Upload a file and attach it

```bash
file_id=$(cademi upload handbook.pdf --purpose pdf --jq .id)
# attachments are keyed by file ID: PUT adds (201) or updates (200) the attachment
cademi content products lessons attachments update prd_12 les_3 "$file_id" -f kind=file -f title="Handbook" --json
cademi download "$file_id" -O handbook.pdf
```

#### Run in CI

```bash
curl -fsSL https://cli.cademi.dev/install.sh | CADEMI_VERSION=0.2.2 CADEMI_NO_MODIFY_PATH=1 bash
export PATH="$HOME/.cademi/bin:$PATH"
export CADEMI_API_KEY="$CADEMI_CI_KEY"                 # ck_test_… for test pipelines
export CADEMI_CLIENT_REQUEST_ID="gh-${GITHUB_RUN_ID}"  # shows up in cademi integrations requests list
cademi integrations credentials current --jq .environment
cademi content products delete prd_old --yes           # confirmations need --yes without a TTY
```

#### Reach an operation newer than the CLI

```bash
cademi doctor                       # warns when the server runs a newer API release
cademi api POST /users --skip-validation -f name="Ana" -f email=ana@example.com -f new_field=value
cademi update                       # then the generated command exists
```

#### Report a bug

```bash
cademi bug --print                                   # prefilled GitHub issue URL (CLI bug)
cademi bug --api --endpoint "PATCH /products/{product_id}" --request-id req_… --print
```

Show the user the link before opening it. The repository `minhacademi/developers` has an `AGENTS.md` with the issue forms; never include secrets or personal data.

### Quick Reference

```bash
cademi auth status --json                              # who am I, which environment
cademi commands --groups --brief --json                # domain index
cademi commands content products --json                # commands of one area
cademi commands users update --schema --json           # body schema
cademi content products list --status published --jq '.[].id'
cademi content products get prd_12 --json
cademi users create -f name="…" -f email=… --json
cademi users update usr_42 -f name="…" --json
cademi content products delete prd_12 --yes
cademi api /credentials/current                        # raw GET
cademi api POST /operations/batches -d @batch.json --wait
cademi operations wait op_… --timeout 2m --json
cademi listen --forward-to localhost:3000/webhooks --json
cademi env                                             # effective settings and their sources
```

### Common Mistakes

- **Guessing body fields**: `-f nickname=ana` exits 2 and lists the accepted fields. Run `cademi commands <cmd> --schema --json` first.
- **Using legacy paths** (`cademi products list`, `cademi credentials update`): they still run but are hidden and unsupported; use `cademi content products list`, `cademi integrations credentials update`.
- **`-f` for booleans and numbers**: `-f send_credentials=false` sends the string `"false"`. Use `-F`.
- **`--all` with `--raw` or `-i`**: exit 2 before any request. Use `--raw` only on single pages.
- **`--json` with `-o table|yaml`**: they conflict; pick one.
- **Deleting or applying in CI without `--yes`**: no terminal means exit 2. Pass `--yes` only when the user asked for an unattended run.
- **Treating `--limit` as a total**: it is the page size. Use `--all` or follow `page.next_cursor`.
- **Offset pagination**: `?offset=` is rejected; the API is cursor-based.
- **Branching on the message**: it is localized. Branch on `error.code` and the exit code.
- **Reading errors from stdout**: errors are on stderr, data on stdout. Capture both in scripts.
- **Assuming exit 1 means the write failed**: a network error after the request was sent leaves the outcome uncertain. Replay with `request.idempotency_key`, do not create a new key.
- **Treating `partially_succeeded` as full success**: exit 0, but some items failed. Check `cademi operations items list <op_id>`.
- **Pre-authenticating**: don't run `cademi auth login` before every command; run the command and act on exit 3.
- **`cademi integrations event-streams events get`**: that is the SSE endpoint; use `cademi listen` instead.
- **Using `cademi api` when a command exists**: generated commands validate input, type the flags and document the body; `cademi api` is the fallback.
- **Pasting secrets or personal data** into issues, chats or commits: only the `request_id` is needed.

## Prerequisites

### Installing the CLI

Check first; install only when it is missing, and tell the user what you are about to run:

```bash
command -v cademi && cademi version          # installed? → cademi 0.2.2 (api 3.10.1, commit …)

# macOS and Linux: no sudo, installs to ~/.cademi/bin and adds it to the shell startup file
curl -fsSL https://cli.cademi.dev/install.sh | bash
export PATH="$HOME/.cademi/bin:$PATH"        # current shell only; new terminals pick it up from the rc file

# Windows PowerShell: installs to %USERPROFILE%\.cademi\bin and updates the user PATH
irm https://cli.cademi.dev/install.ps1 | iex

# CI or a container: pin the version and leave startup files alone
curl -fsSL https://cli.cademi.dev/install.sh | CADEMI_VERSION=0.2.2 CADEMI_NO_MODIFY_PATH=1 bash

cademi version                               # verify; the script already checked the SHA-256
cademi update --check                        # later updates are signed (ed25519) and verified
```

- If `cademi` is not found right after installing, the current shell has not reloaded its PATH: export it as above or open a new terminal.
- The installer needs `curl` and network access to `cli.cademi.dev`; it never asks for sudo. `CADEMI_INSTALL_DIR` changes the destination.
- `CI` or `CADEMI_DISABLE_AUTOUPDATE` turn automatic updates off; `cademi update --version 0.2.2` pins or rolls back.
- Installing the CLI does not sign anyone in: the next step is a credential (below).
- `cademi skills install` installs this skill, at the version bundled with the installed CLI, into `~/.claude/skills` and `~/.agents/skills`. Without `--agent` or `--dir` it only installs for agents whose directory already exists, so an agent that never ran on the machine does not get it; `--dir .claude/skills` installs into one project, and `cademi skills status` checks the copy. After the CLI is updated, the first command you run refreshes the copies in `~/.claude/skills` and `~/.agents/skills`; a copy installed with `--dir` is not refreshed: run `cademi skills install --dir <path>` again.
- `cademi mcp install <client>` adds the Cademí MCP server to an assistant (`claude`, `cursor`, `vscode`, `copilot`, `codex`, `gemini`, `opencode`); `cademi mcp docs` has the details. Prefer the MCP when the assistant should call the API as tools inside a conversation; prefer the CLI for scripts, CI, bulk work, events, and files.

### Credentials

Credentials are created in the Cademí dashboard (`ck_live_…` for production, `ck_test_…` for the sandbox) with a policy that grants permissions such as `products.read` or `users.create`. Each command's catalog entry names the permission it needs. Secrets live only in the OS keychain (service `cademi-cli`) or in `CADEMI_API_KEY`; profiles without secrets are in `config.toml` under `cademi env config_dir`.

## Command Reference

One row per reference file. Each file carries the signature, route, permission, arguments, flags, body fields and examples of its commands for cademi 0.2.2. The live catalog is always `cademi commands <prefix> --json`.

<!-- commands:start -->

| Reference | Prefix | Commands | Covers |
|---|---|---|---|
| `references/core.md` | `cademi` | 24 | api, auth, bug, commands, config, doctor, download, env, listen, mcp, profiles, skills, update, upload, version |
| `references/guides.md` | `cademi guide` | 6 | errors, input, output, reordering, retries |
| `references/account.md` | `cademi account` | 18 | administrators, audit-entries, capabilities, domains, get, replicas, usage |
| `references/automations.md` | `cademi automations` | 18 | diamonds |
| `references/content-banners.md` | `cademi content banners` | 7 | copies, create, delete, get, list, order, update |
| `references/content-products.md` | `cademi content products` | 15 | certificate, comments, content, copies, create, delete, get, list, order, publish-all, questions, update |
| `references/content-products-access-schedules.md` | `cademi content products access-schedules` | 10 | create, delete, get, list, rules, update |
| `references/content-products-exams.md` | `cademi content products exams` | 19 | answer-key, attempts, create, delete, get, list, questions, results, update |
| `references/content-products-lessons.md` | `cademi content products lessons` | 16 | attachments, connection, content, copies, create, delete, get, list, taxonomy-terms, update |
| `references/content-products-modules.md` | `cademi content products modules` | 9 | content, copies, create, delete, get, list, publish-all, update |
| `references/content-products-taxonomies.md` | `cademi content products taxonomies` | 10 | create, delete, get, list, terms, update |
| `references/content-showcases.md` | `cademi content showcases` | 8 | copies, create, delete, get, list, order, products, update |
| `references/files.md` | `cademi files` | 14 | delete, download-links, exports, get, list, uploads |
| `references/gamification.md` | `cademi gamification` | 2 | rankings, scores |
| `references/integrations.md` | `cademi integrations` | 10 | event-streams, events, requests, schema |
| `references/integrations-credentials.md` | `cademi integrations credentials` | 15 | create, current, current-policies, delete, get, list, permission-catalog, policies, policy-templates, secret-rotations, update |
| `references/integrations-webhooks.md` | `cademi integrations webhooks` | 13 | create, delete, deliveries, get, list, replays, secret-rotations, update |
| `references/operations.md` | `cademi operations` | 9 | attempts, batches, get, items, list, update, wait |
| `references/reports.md` | `cademi reports` | 13 | activity, certificates, email-bounces, enrollments, exams, exports, lessons, list, products, rankings, support, users |
| `references/sales.md` | `cademi sales` | 21 | deliveries, events, gateways, transactions |
| `references/sandbox.md` | `cademi sandbox` | 7 | get, reset, resets, run, test-scenarios |
| `references/settings-access.md` | `cademi settings` | 8 | authentication, registration, security, user-profile |
| `references/settings-definitions.md` | `cademi settings` | 18 | configuration, custom-code, custom-fields, tags |
| `references/settings-platform.md` | `cademi settings` | 14 | admin-area, app, branding, embedded-pages, gamification, platform, sharing |
| `references/settings-emails.md` | `cademi settings emails` | 7 | get, templates, update |
| `references/settings-legal-terms.md` | `cademi settings legal-terms` | 6 | acceptances, create, get, list, update |
| `references/settings-menus.md` | `cademi settings menus` | 8 | get, items, list |
| `references/settings-support.md` | `cademi settings support` | 8 | comments, faq, get, questions, update |
| `references/support.md` | `cademi support` | 5 | departments |
| `references/support-comments.md` | `cademi support comments` | 8 | delete, get, list, replies, update |
| `references/support-faqs.md` | `cademi support faqs` | 6 | create, delete, get, list, order, update |
| `references/support-questions.md` | `cademi support questions` | 9 | delete, get, list, replies, update |
| `references/support-tickets.md` | `cademi support tickets` | 9 | create, get, list, replies, update |
| `references/users-learning.md` | `cademi users` | 13 | certificates, enrollments, progress, score-adjustments, scores |
| `references/users-profile.md` | `cademi users` | 17 | attribution, avatar, custom-fields, notes, tags, term-acceptances |
| `references/users.md` | `cademi users` | 9 | access, access-emails, activity, create, delete, get, list, password-reset-emails, update |
| `references/users-imports.md` | `cademi users imports` | 10 | analyses, create, delete, get, list, processing-attempts, rows, update |
| `references/users-products.md` | `cademi users products` | 9 | access, list, progress |

<!-- commands:end -->

## Global Flags and Environment

| Flag | Effect |
|---|---|
| `-p, --profile <name>` | Use this saved connection (also `CADEMI_PROFILE`) |
| `--base-url <url>` | API URL instead of the profile's (also `CADEMI_BASE_URL`; default `https://api.cademi.com.br`) |
| `--debug` | Log each HTTP request to stderr with method, URL, status, duration and `request_id`; never secrets |

| Variable | Effect |
|---|---|
| `CADEMI_API_KEY` | Credential secret, autonomous mode; overrides every profile |
| `CADEMI_ACCESS_TOKEN` | Delegated token from the Cademí MCP; cannot be combined with `CADEMI_API_KEY` |
| `CADEMI_PROFILE`, `CADEMI_BASE_URL`, `CADEMI_CONFIG_DIR` | Profile, API URL and configuration directory |
| `CADEMI_LANG` | Language of API messages (`pt`, `en`, `es`, `fr`); defaults to the system locale |
| `CADEMI_CLIENT_REQUEST_ID` | Your run identifier, sent as `X-Client-Request-Id` and recorded in the request log |
| `CADEMI_MAX_RETRIES`, `CADEMI_MAX_RETRY_WAIT` | Retries per request (default 3; `0` disables) and the longest accepted wait (default `60s`) |
| `CADEMI_LISTEN_SECRET` | Signing secret for `cademi listen --forward-to` when there is no profile |
| `CADEMI_DISABLE_AUTOUPDATE`, `CI` | Turn automatic updates off |
| `NO_COLOR` | Turn colors off (`TERM=dumb` too) |

## Output Formats

| Flag | Output |
|---|---|
| (none) | Table for lists on a terminal; JSON when piped or for single objects |
| `--json` / `-o json` | JSON of the response `data` |
| `-o yaml`, `-o table` | YAML or table |
| `--jq '<expr>'` | Filtered JSON; strings unquoted; overrides `-o` |
| `--raw` | Full envelope (`data`, `page`, …) |
| `-i, --include` | HTTP status and headers before the body (not a JSON document) |

## Further Reading

- CLI docs: https://cademi.dev/cli/overview.md, commands.md, api.md, listen.md, config.md, sandbox.md, files.md, ci.md, exit-codes.md, environment.md
- API docs index for LLMs: https://cademi.dev/llms.txt
- Issues and discussions: https://github.com/minhacademi/developers (read its `AGENTS.md` before opening one)
