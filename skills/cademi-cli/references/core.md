---
name: cademi-cli-core
description: "Built-in commands: authentication and profiles, raw API requests, events, declarative configuration, files, diagnostics and updates"
metadata:
  cademi-cli: "0.3.1"
  cademi-api: "3.13.1"
---

# Core Commands

> cademi 0.3.1, API 3.13.1. The live catalog is always `cademi commands <prefix> --json`.

Commands that are not generated from the API contract. Resource commands (`cademi <domain> ...`) live in the other reference files; `cademi guide` topics are in `guides.md`.

## Shared flag sets

Each command lists the sets it accepts. The flags of a set are:

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

### `cademi api [method] <path>`

Make an authenticated request to any API v3 endpoint

Make an authenticated request to the Cademí API v3 and print the response.

The path is relative to /api/v3. The method defaults to GET, or POST when --data is given.
For GET requests, -f/-F become query parameters; otherwise they build the JSON body.
Internal retries reuse the same Idempotency-Key. Each new invocation generates a
new key. A failed HTTP attempt returns the key with the error (request.idempotency_key
in JSON). To repeat that request, pass the returned key with --idempotency-key
and the same body; see cademi guide retries --help.

Before sending, the request is checked against the API contract embedded in this
version (api 3.13.1): unknown query parameters and body fields are
rejected with exit 2. Use --skip-validation to call something newer than that release.

**Arguments:**
- `method` (optional)
- `path`

**Flag sets:** output-basic, body, idempotency, async

**Flags:**
- `--all` — Collect remaining pages into one array in memory (conflicts with --raw and --include; --jq runs after collection)
- `-H, --header <stringArray>` — Add a request header: 'Name: value'
- `--if-match <string>` — Send If-Match with this ETag (fails with 412 if the resource changed)
- `-i, --include` — Print HTTP status and headers before the body on stdout (output is not a JSON document) (conflicts with `--all`)
- `--raw` — Print the full response envelope instead of data (conflicts with `--all`)
- `--skip-validation` — Send query parameters and body fields that the embedded API contract does not know

**Examples:**

```bash
cademi api /credentials/current
cademi api /products -f status=published --all --jq '.[].id'
cademi api POST /users -f name="Ana Souza" -f email=ana@example.com
cademi api PATCH /products/prd_12 --if-match 'W/"product:prd_12:..."' -f status=draft
cademi api POST /operations/batches -d @batch.json --wait
```

### `cademi bug`

Report a bug in the CLI or the API: open a GitHub issue already filled in

Open a new issue on github.com/minhacademi/developers, the public tracker for the
Cademí API v3 and the CLI. Nothing is sent until you review the issue and submit it
in the browser.

By default it opens the CLI bug report with the CLI version, platform and the output
of `cademi env` filled in. With --api it opens the API bug report instead, for a
response that contradicts the documentation; pass --endpoint and --request-id (from
the error or the X-Request-Id header) to fill them in.

The environment never includes secrets. Issues are public: do not paste API keys,
tokens or personal data. Report security vulnerabilities privately (see SECURITY.md
in that repository), and questions about your account or data to Cademí support.

**Flags:**
- `--api` — Report an API bug (the response contradicts the documentation) instead of a CLI bug
- `--endpoint <string>` — With --api: method and path, e.g. "GET /api/v3/products"
- `--print` — Print the issue URL instead of opening the browser
- `--request-id <string>` — The request_id of the API call (error body or X-Request-Id header)
- `--title <string>` — Issue title

**Examples:**

```bash
cademi bug
cademi bug --title "products list --all stops after the first page"
cademi bug --api --endpoint "PATCH /api/v3/products/{product_id}" --request-id 01J8Z3ZQ4H8K2M0T1S9P7YQF5C
cademi bug --print
```

### `cademi commands <prefix...>`

Discover domains and commands, with API routes, arguments and flags

List every command of the CLI in one call. For commands generated from the API,
each entry includes the operationId, the HTTP method and route, and the required
permission. Global flags and the flag sets shared by many commands (output, body,
async, pagination…) are described once; each command lists the sets it accepts
and its own flags. Entries use canonical command paths; legacy_paths records
compatible older paths. Prefix filtering accepts canonical and legacy paths.
Groups describe resource relationships. Use --groups for immediate child groups.

Useful for scripts and AI agents: `cademi commands --json` describes the whole
CLI without running --help on each command. Start with
`cademi commands --groups --brief --json` for a compact domain index,
then narrow the prefix with --groups before requesting individual commands.
Use `cademi commands content products --json` for full detail in one area.
Use --schema for complete OpenAPI request bodies and their referenced schemas,
including field descriptions, constraints and nested properties. This inspection
uses the bundled contract and makes no API requests. JSON is the default for --schema.
--schema cannot be combined with --brief or --groups and requires JSON or YAML.
--builtin cannot be combined with --generated or --groups.

**Arguments:**
- `prefix` (optional) (repeatable) — Command path prefix, supplied as separate words; accepts canonical and legacy paths.

**Flag sets:** output-basic

**Flags:**
- `--brief` — Only command, route, summary and argument names; combine with --groups for a compact index (conflicts with `--schema`)
- `--builtin` — Only built-in commands (auth, api, listen, config…) (conflicts with `--generated`, `--groups`)
- `--generated` — Only commands generated from the API (conflicts with `--builtin`)
- `--groups` — List immediate resource groups below the prefix instead of commands (conflicts with `--builtin`, `--schema`)
- `--schema` — Include complete OpenAPI input schemas and field context (JSON by default) (conflicts with `--brief`, `--groups`)

**Examples:**

```bash
cademi commands
cademi commands --groups --brief --json
cademi commands content products --groups --brief --json
cademi commands content products --json
cademi commands settings support comments update --schema --json
cademi commands settings support comments update --schema --json --jq '{request_body: .commands[0].request_body, components: .components}'
cademi commands --json --jq '.commands[] | select(.method == "DELETE") | .command'
```

### `cademi doctor`

Diagnose configuration, connectivity, credentials and API compatibility

Diagnose configuration, connectivity, credentials and API compatibility

**Flag sets:** output-basic

### `cademi download <file_id>`

Download a file through a short-lived link

Download a file through a short-lived link

**Arguments:**
- `file_id`

**Flags:**
- `-O, --output-file <string>` — Where to save the file, or - for stdout (default: the file name)

**Examples:**

```bash
cademi download file_01J8Z3...
cademi download file_01J8Z3... -O handbook.pdf
cademi download file_01J8Z3... -O - > handbook.pdf
```

### `cademi env [name]`

Show the effective configuration and where each value comes from

Show every setting the CLI is using, its value and its source:
flag, environment variable, config file, profile or default. Secrets are never
printed, only whether they are set and where.

Precedence, highest first: command-line flag, environment variable, config file
(or the active profile), built-in default.

With a name, print only that value (like `go env GOPATH`).

**Arguments:**
- `name` (optional)

**Flag sets:** output-basic

**Examples:**

```bash
cademi env
cademi env base_url
cademi env --json
```

### `cademi listen`

Stream account events to the terminal or forward them to a local endpoint

Stream the public events of the account in real time (Server-Sent Events).

listen creates a temporary event stream (unless --stream is given), prints each
event and, with --forward-to, POSTs it to a local endpoint in the same format
and with the same signature scheme as a webhook delivery:

  Body:    {"id","type","version","occurred_at","data","delivery_id","attempt"}
  Headers: Cademi-Signature: t=<unix>,v1=<hex HMAC-SHA256(secret, t + "." + body)>
           Cademi-Signature-Version: 1, Cademi-Webhook-Id: local,
           Cademi-Delivery-Id, Cademi-Event-Type

The signing secret (whsec_local_...) is generated once per profile and kept in
the keychain, so your receiver can verify signatures while you develop.
Use --print-secret to show it. On Ctrl-C the temporary stream is revoked.

In a sandbox, run a test scenario in another terminal to produce events:
  cademi sandbox run <scenario_id>

**Flags:**
- `--events <stringSlice>` — Only these event types (e.g. product.created,user.updated)
- `--forward-to <string>` — POST each event to this URL (e.g. localhost:3000/webhooks)
- `--json` — Print each event as one JSON line
- `--keep` — Do not revoke the temporary event stream on exit
- `--last-event-id <string>` — Resume after this event ID
- `--print-secret` — Print the local signing secret and exit
- `--replay` — Also deliver the retained events published before listen started
- `--resources <stringSlice>` — Only events about these resource types (e.g. product,user)
- `--stream <string>` — Use an existing event stream instead of creating a temporary one

**Examples:**

```bash
cademi listen
cademi listen --events product.created,product.updated --forward-to localhost:3000/webhooks/cademi
cademi listen --json | jq -c 'select(.type=="user.created")'
```

### `cademi update`

Update cademi to the latest version

Download and install the latest cademi release for this platform.

Releases are verified before installing: the checksums file must carry a valid
ed25519 signature from the Cademí release key embedded in this binary, and the
package must match its SHA-256.

By default cademi also updates itself in the background, at most once a day, and
tells you on the next run. Turn that off with `cademi update --auto off` or
CADEMI_DISABLE_AUTOUPDATE=1 (it is always off when CI is set).
The first run of a new version also refreshes the cademi-cli agent skill where
it is already installed (see `cademi skills`).
Installed with Homebrew? Use `brew upgrade cademi` instead.

**Flags:**
- `--auto <string>` — Turn automatic updates on or off
- `--channel <string>` — Set the update channel: stable or beta
- `--check` — Only check whether a newer version exists
- `--force` — Reinstall even if up to date, or replace a development/Homebrew build
- `--version <string>` — Install this version instead of the latest (also to roll back)

**Examples:**

```bash
cademi update
cademi update --check
cademi update --version 0.3.1      # install a specific version (also to roll back)
cademi update --channel beta
cademi update --auto off
```

### `cademi upload <file>`

Upload a file and print the resulting file object

Upload a file through an upload session: the CLI opens the session with the
file's SHA-256, sends the parts in parallel straight to storage through
presigned URLs and completes it. Use the returned file ID (file_...) in the
resource that needs it (lesson attachment, showcase image, import spreadsheet…).

**Arguments:**
- `file`

**Flag sets:** output-basic

**Flags:**
- `--content-type <string>` — Content type (default: detected)
- `--name <string>` — File name to record (default: the local file name)
- `--parallel <int>` — Parts uploaded in parallel
- `--purpose <string>` — Upload purpose: image, pdf, import, editor or document (required)

**Examples:**

```bash
cademi upload handbook.pdf --purpose pdf
cademi upload students.xlsx --purpose import --jq .id
```

### `cademi version`

Print the CLI and API contract versions

Print the CLI and API contract versions

### `cademi auth login`

Connect the CLI to a Cademí account

Connect the CLI to a Cademí account and save the connection as a profile.

By default the login is in human mode: the browser opens, you enter your
account's address, sign in there as an administrator with two-factor
authentication and choose Authorize (OAuth with PKCE). Then you paste the secret
of an API credential linked to that administrator. Every call carries both, and
the audit trail records the administrator as the author.

--platform takes your account's address (acme.cademi.com.br, a custom domain
or just its subdomain) and skips the question in the browser. It is saved in the
profile and reused on the next login.

With --api-key-only the profile uses only the API credential (autonomous mode).

Secrets are stored in the operating system keychain, never in files.

**Flags:**
- `--api-key-only` — Use only the API credential (autonomous mode), without OAuth
- `--no-browser` — Print the authorization URL instead of opening the browser
- `--platform <string>` — Your account's address (acme.cademi.com.br, a custom domain or its subdomain); saved in the profile
- `--with-key` — Read the API key secret from stdin

**Examples:**

```bash
cademi auth login
cademi auth login --platform acme.cademi.com.br
cademi auth login --profile sandbox --base-url https://api.cademi.com.br
echo "$KEY" | cademi auth login --api-key-only --with-key
```

### `cademi auth logout`

Revoke the OAuth session and remove the profile's secrets

Revoke the OAuth tokens of the profile (RFC 7009) and remove its secrets from the keychain.

The API credential itself is not revoked: it may be shared with other people or systems.
Revoke it in the dashboard if needed, or with `cademi integrations credentials update <credential_id> -F status=revoked`
using another credential that has the credentials.manage permission.

### `cademi auth status`

Show the active profile and validate its credential

Show the active profile and validate its credential

**Flag sets:** output-basic

### `cademi auth switch <profile>`

Switch the current profile (same as `cademi profiles use`)

Switch the current profile (same as `cademi profiles use`)

**Arguments:**
- `profile`

### `cademi config apply <manifest>`

Plan and apply a manifest

Compute the plan for a manifest, show it, ask for confirmation and apply it.
The API applies the plan as an asynchronous operation, one item per step; the
command waits for it unless --no-wait is given.

With --plan, apply a plan saved by `cademi config plan --out` instead. The manifest is
still needed when the plan has secrets (***), to fill in their real values.

**Arguments:**
- `manifest`

**Flag sets:** output-basic, confirm

**Flags:**
- `--no-wait` — Do not wait for the operation to finish
- `--plan <string>` — Apply a plan `file` saved with cademi config plan --out

**Examples:**

```bash
cademi config apply catalog.yaml
cademi config plan catalog.yaml --out plan.json && cademi config apply --plan plan.json catalog.yaml
```

### `cademi config plan <manifest>`

Show the changes a manifest would make

Compute a signed plan with the differences between the manifest and the account.
Nothing is changed. The plan is valid for 24 hours; save it with --out and apply
it later with `cademi config apply --plan <file> <manifest>`.

**Arguments:**
- `manifest`

**Flag sets:** output-basic

**Flags:**
- `--out <string>` — Save the plan (JSON) to this file

### `cademi config validate <manifest>`

Check a manifest without changing anything

Check a manifest without changing anything

**Arguments:**
- `manifest`

**Flag sets:** output-basic

### `cademi mcp docs`

List the Cademí MCP documentation

Print links to the Cademí MCP documentation, with a one-line summary of each page.

### `cademi mcp install <claude|cursor|vscode|copilot|codex|gemini|opencode>`

Add the Cademí MCP server to an AI assistant

Add the Cademí MCP server to an AI assistant's configuration, as the remote
server "cademi" (Streamable HTTP, OAuth). Other servers and settings in that
configuration are kept. Sign-in happens in the assistant afterwards; the next
step is printed when the server is added.

Clients:
  claude    Claude Code (its CLI)
  cursor    Cursor (its MCP config file)
  vscode    VS Code (its MCP config file)
  copilot   GitHub Copilot CLI (its MCP config file)
  codex     Codex (its CLI)
  gemini    Gemini CLI (its MCP config file)
  opencode  OpenCode (its MCP config file)

Without --project the server is added for your user, in every project. An
existing "cademi" entry pointing elsewhere is replaced only with --force.

**Arguments:**
- `claude|cursor|vscode|copilot|codex|gemini|opencode`

**Flags:**
- `--force` — Replace an existing cademi entry that points to another URL
- `--project` — Add the server to this project only (config file in the current directory)
- `--url <string>` — MCP server URL

**Examples:**

```bash
cademi mcp install claude
cademi mcp install codex
cademi mcp install vscode --project    # .vscode/mcp.json in this directory
```

### `cademi profiles list`

List saved profiles

List saved profiles

**Aliases:**

```text
ls
```

**Flag sets:** output-basic

### `cademi profiles use <name>`

Set the current profile

Set the current profile

**Aliases:**

```text
switch
```

**Arguments:**
- `name`

### `cademi skills install`

Install or update the cademi-cli skill in agent skill directories

Install the cademi-cli skill bundled with this binary. An existing copy is
replaced as a whole, so files dropped by the new version disappear.

Without --agent or --dir, installs for every agent whose directory exists
(~/.claude, ~/.agents) and never creates those directories. --agent installs
even when the directory is missing. CLAUDE_CONFIG_DIR moves the Claude Code
directory.

**Flag sets:** output-basic

**Flags:**
- `--agent <stringSlice>` — Install for this agent: claude (~/.claude/skills), agents (~/.agents/skills: Codex, Cursor, Gemini CLI, OpenCode, GitHub Copilot and others) or all
- `--dir <stringSlice>` — Install into this skills directory (the skill goes in <dir>/cademi-cli)
- `--force` — Replace a cademi-cli directory that is not this skill

**Examples:**

```bash
cademi skills install
cademi skills install --agent all
cademi skills install --dir ~/work/repo/.agents/skills
cademi skills install --json
```

### `cademi skills status`

Show where the cademi-cli skill is installed and whether it is current

Show each agent skill directory, whether the cademi-cli skill is installed
there and whether it matches the version bundled with this binary.

Status values: up-to-date, outdated (another version), modified (same version,
edited files), not-installed, not-detected (the agent directory does not exist)
and foreign (a cademi-cli directory that is not this skill).

**Flag sets:** output-basic

**Flags:**
- `--agent <stringSlice>` — Only this agent: claude (~/.claude/skills), agents (~/.agents/skills: Codex, Cursor, Gemini CLI, OpenCode, GitHub Copilot and others) or all
- `--dir <stringSlice>` — Check this skills directory too

**Examples:**

```bash
cademi skills status
cademi skills status --json
```

### `cademi skills uninstall`

Remove the cademi-cli skill from agent skill directories

Remove the cademi-cli skill from ~/.claude/skills and ~/.agents/skills (or
the agents and directories you name). Directories named cademi-cli that are not
this skill are left alone.

**Flags:**
- `--agent <stringSlice>` — Only this agent: claude (~/.claude/skills), agents (~/.agents/skills: Codex, Cursor, Gemini CLI, OpenCode, GitHub Copilot and others) or all
- `--dir <stringSlice>` — Remove from this skills directory

**Examples:**

```bash
cademi skills uninstall
cademi skills uninstall --agent claude
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
