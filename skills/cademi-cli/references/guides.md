---
name: cademi-cli-guides
description: "Automation guides built into the CLI (errors, input, output, reordering, retries), read offline with --help"
metadata:
  cademi-cli: "0.2.6"
  cademi-api: "3.12.1"
---

# Guide Commands

> cademi 0.2.6, API 3.12.1. The live catalog is always `cademi commands <prefix> --json`.

`cademi guide <topic> --help` prints these guides locally, without contacting the API. They are reproduced here so an agent can read them without running the CLI.

## `cademi guide errors --help`

Exit codes, error envelopes and asynchronous outcomes.

**Exit Codes:**

```text
0    Completed successfully (also partially_succeeded operations).
1    Other failures, including transport errors, local wait timeouts,
     failed/canceled server operations, and output processing errors.
2    CLI usage or local request validation error.
3    Missing/invalid credentials, reauthentication required, or API 401.
4    API 403 (permission denied).
5    API 404 (not found).
6    Other API 4xx, including 409, 412 and 422.
7    API 429 after retries.
8    API 5xx after retries.
130  Local cancellation (Ctrl-C or a declined confirmation).
```

**Streams And Errors:**

```text
Results go to stdout; errors, warnings and progress go to stderr.
Request JSON errors with --json, --output json or --jq. API errors retain
their API error fields; local errors use {"error":{"code":"cli_error","message":"..."}}.
Failed HTTP attempts with an idempotency key add request.method, request.path
and request.idempotency_key alongside error. Exit codes remain unchanged.
Warnings/progress can accompany the error on stderr: the whole stream is not
guaranteed to be one JSON document. Cancellation emits no JSON error body
unless an HTTP attempt with an idempotency key needs to be reported.
Flag parsing and local request validation happen before the resource request.
Workflow checks may contact the API before returning a usage error.
A transport error can leave a write outcome uncertain. Do not infer success
solely from stdout; check the exit status.
```

**Async Operations:**

```text
--wait and operations wait print the terminal operation even if it failed
or was canceled, then exit 1. partially_succeeded exits 0: inspect status and
cademi operations items <operation_id> for individual item failures.
--wait-timeout and operations wait --timeout accept durations such as 30s or
2m (0 means no limit). A timeout exits 1 and Ctrl-C exits 130; neither cancels
the operation on the server. Resume with operations wait <operation_id>.
Polling failures use the API/transport codes above.
```

**Examples:**

```bash
cademi operations wait op_42 --timeout 2m --json
cademi operations items op_42 --all --json
```

## `cademi guide input --help`

JSON bodies, field types and merge precedence.

**Body Merging:**

```text
The base is --data (JSON literal, @file, or @- for stdin). Then all -f fields
are applied in their own order, then all -F fields in their own order,
regardless of how -f and -F were interleaved on the command line.
Repeated scalar paths overwrite; a.b and a[b] address nested fields;
a[] appends to an array. Arrays of objects must use --data or a JSON array.
Combining -f/-F with --data requires a JSON object as the base.
```

**Types And Quoting:**

```text
-f always supplies a string in a body (including literal @file).
-F recognizes booleans, null, numbers, JSON objects/arrays/quoted strings;
other values remain strings. -F key=@file reads file contents as a string,
not as JSON. --data @file parses the entire file as JSON. @- reads stdin.
Quote JSON and bracketed keys to protect them from shell expansion.
Null is an explicit value, not omission; consult the operation's schema.
For cademi api GET, -f/-F are repeated query values (no JSON type inference);
@file/@- supplies text, and --data is rejected.
```

**Examples:**

```bash
cademi users update usr_42 --data @user.json -f name=First -f name=Final
cademi users update usr_42 -F 'settings={"block_comments":true,"block_questions":false,"block_support":false}'
cademi users create -f name=Ana -f email=ana@example.com -f 'tags[]=tag_1' -f 'tags[]=tag_2'
cademi users create --data '{"name":"Ana","email":"ana@example.com","custom_fields":{"cfd_1":"Engineer"}}'
cademi settings custom-fields list --json
cademi commands users update --schema --json
```

## `cademi guide output --help`

Pagination, JSON, jq and stdout/stderr.

**Output:**

```text
Resource commands print the response data by default. --json selects JSON;
without a format, lists use tables on a terminal and JSON when redirected.
--raw --json prints the full API envelope, including page.next_cursor.
--include prepends HTTP status and headers to stdout, including ETag; this
output is not a JSON document even with --json. Diagnostics go to stderr.
With --wait, the final operation is printed directly, even with --raw;
--include shows headers of the original response, not the final poll.
```

**Pagination:**

```text
--limit controls each page, not the total. Bounds and API defaults come from
the OpenAPI parameters in each command's help; omitted flags stay omitted.
Pass page.next_cursor to --cursor with the same scope, filters and sort.
Stop when next_cursor is null or empty. Cursors are opaque: do not construct
them or assume a numeric offset. Use a fresh list if a cursor is rejected.
--all follows remaining pages, starting at --cursor when supplied, and buffers
one array in memory. It only sees resources granted to the credential.
--all conflicts with --raw and --include (exit 2, before any request).
If fetching any page fails, no partial collection is printed to stdout.
```

**Jq:**

```text
--jq receives data, or the full envelope with --raw. With --all it runs once
after aggregation. It overrides --output: strings print unquoted; other values
print as JSON, one result per line. Use an array expression for one JSON value.
--json still conflicts with --output table/yaml, including when --jq is set.
A jq runtime error can occur after earlier jq results have been printed.
```

**Examples:**

```bash
cademi users list --limit 50 --raw --json
cademi users list --cursor '<page.next_cursor>' --raw --json
cademi users list --all --jq '[.[] | {id, name}]'
cademi support tickets replies list tkt_42 --all --jq '.[-20:]'
  The last example fetches every remaining message; jq only reduces output.
  For bounded exploration, use --limit 20 --raw --json and follow cursors.
```

## `cademi guide reordering --help`

Collect complete IDs, inspect revisions and submit a new order.

**Workflow:**

```text
Read the target order update --help for the API's scope and completeness rules.
Start a fresh list without cursor or narrowing filters such as status, kind
or search; keep the parent/container scope. --all does not expand permissions.
Credentials must be able to discover the entire set required by the operation.
Review and rearrange IDs in the saved file before submitting it.

PRODUCT CONTENT EXAMPLE (shell)
cademi content products get prd_42 --include --json
# Save the exact ETag header, including its quotes, before collecting IDs.
cademi content products content list prd_42 --all --jq '{ids: map(.id)}' > order.json
# Edit order.json to the desired order. Then send that file and the saved ETag:
cademi content products content order update prd_42 --data @order.json --if-match '"<saved-etag>"'
# An inline array must be shell-quoted, e.g. --data '{"ids":["mod_2","mod_1"]}'.

OTHER COLLECTIONS (collect with --all --jq '{ids: map(.id)}' > order.json)
Module: cademi content products modules content list prd_42 mod_7
  Revision: cademi content products modules get prd_42 mod_7 --include
  Write: cademi content products modules content order update prd_42 mod_7 --data @order.json --if-match '"<saved-etag>"'
Banners: cademi content banners list
  Write: cademi content banners order update --data @order.json
Showcases: cademi content showcases list
  For a group's children, add --parent-id shw_7 when listing and include
  "parent_id":"shw_7" alongside ids in order.json.
  Write: cademi content showcases order update --data @order.json
Showcase products: cademi content showcases products list shw_42
  Revision: cademi content showcases get shw_42 --include
  Write: cademi content products order update shw_42 --data @order.json --if-match '"<saved-etag>"'
Exam questions: cademi content products exams questions list prd_42 exm_7
  Revision: cademi content products exams get prd_42 exm_7 --include
  Write: cademi content products exams questions order update prd_42 exm_7 --data @order.json --if-match '"<saved-etag>"'
```

**Recovery:**

```text
On order_set_mismatch, check scope/visibility and fetch the complete set again.
On 412, retrieve the new ETag and collection, review changes, then rebuild the
intended order. Do not blindly retry stale IDs. A changed request needs a new
idempotency key; see cademi guide retries --help.
```

## `cademi guide retries --help`

Idempotency across retries and separate invocations.

**Write Retries:**

```text
POST, PATCH, PUT and DELETE receive an Idempotency-Key. Internal retries of
one request reuse the same key and body. Starting the command again generates
a new key unless --idempotency-key is supplied: it is a new API request intent.
If an HTTP attempt fails, the error on stderr includes the actual key used,
including on transport failures without an API response. JSON errors add:
"request":{"method":"POST","path":"/api/v3/users","idempotency_key":"..."}
To repeat it, pass request.idempotency_key with --idempotency-key and keep
the same method, path and body in the same account/credential context.
Do not reuse a key for a changed body or an unrelated operation. API retention
and replay rules still apply; a network failure does not prove a write failed.
Local validation errors before an HTTP attempt have no request key. A failure
while polling an accepted async operation refers to the poll, not the write;
resume with operations wait instead of creating the operation again.
```

**Example:**

```text
cademi users create --data @new-user.json --json
# If it fails, copy request.idempotency_key from the JSON error on stderr:
cademi users create --data @new-user.json --idempotency-key '<returned-key>' --json

You can still choose and save an explicit key before the first attempt when
you need recovery after process termination or lost error output. No error
can be printed if the process is forcibly killed.

Rate limits, eligible server errors and transport failures can be retried
internally. After retries are exhausted, use the exit code and error body;
see cademi guide errors --help.
```

---

All commands also accept `--help` and the global flags `-p/--profile`, `--base-url` and `--debug`. With `--json` or `--jq`, errors are printed as JSON on stderr: branch on `error.code`, never on the message. See `SKILL.md` for exit codes and the discovery flow.
