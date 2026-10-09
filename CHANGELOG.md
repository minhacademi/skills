# Changelog

All notable changes to the skills in this repository. Each skill carries its own version in the `metadata` block of its `SKILL.md`.

## cademi-cli 0.1.10 (2026-10-09)

- References follow the `cademi` CLI 0.3.1 and API 3.13.1, a documentation-only API release: requests, responses and commands do not change.
- `cademi integrations event-streams create` states the limit of 5 active streams per credential and 20 per account. `cademi certificates create` lists the `details[].reason` values of the 409 (`no_access`, `template_disabled`, `exam_not_passed`, `not_completed`).
- The declarative manifest guidance says `module` and `lesson` require `product_id` in `attributes` (it accepts `$ref`), that `lesson` takes `module_id` and `content`, and that the validation report lists the manifest's own codes and always returns an empty `warnings`.

## cademi-cli 0.1.9 (2026-10-09)

- References follow the `cademi` CLI 0.3.0 and API 3.13.0 (includes 3.12.2).
- First import of an account: until the account has a processed import, `cademi users imports processing-attempts create` with `mode=initial` returns `409 feature_disabled` with `details[].reason` `first_import_requires_release` (exit code 6). Contact Cademí support to release it.
- `POST /event-streams` returns the standard error envelope on `429 too_many_streams`, without the rate-limit fields; `cademi listen` already handled it.
- Errors in the terminal show each `details` item with its reason next to the code, for example `(feature_disabled, reason: first_import_requires_release)`. The `--json` envelope and exit codes do not change.
- Descriptions updated for `users imports processing-attempts create`, `automations diamonds delete`, `files exports create` and `Certificate.reissue`; `null` is part of the enum of `ReleaseRule.type` and `ProductAccess.denial_reason`; every operation declares 403 and 503.
- API fixes that need no CLI change: `files exports create` and import analyses work again; `files exports list` and `get` and `reports exports get` skip records without any date; `integrations webhooks get` on a deleted endpoint is 404; `sales deliveries update` is idempotent when archiving or reactivating; an administrator granted by ids manages the ones in scope (out of scope is 404); `gamification scores list` without filters no longer times out; correcting the reply of a reopened question moves it back to answered; the user of a ticket without a user is null.

## cademi-cli 0.1.8 (2026-10-07)

- References follow the `cademi` CLI 0.2.7 and API 3.12.1.
- The page the browser shows after `cademi auth login` now uses the cademi.dev look, with the Cademí symbol and one color per outcome (green authorized, red denied, amber when the authorization comes back twice).
- The `auth login` text, the root help, the login messages and the browser pages say "account" instead of "platform" and "instance", matching the portal. The `--platform` flag, the `platform` key in `--json` and the `platform_mismatch` code are unchanged.
- Examples include required flags (`cademi account usage --period 30d`, `reports activity list`, `reports lessons list`, `reports support get`), and there is a new "Account overview" recipe.

## cademi-cli 0.1.7 (2026-10-06)

- References follow the `cademi` CLI 0.2.6 (API 3.12.1, compatible with 3.12.0): the API spec now describes what it already returned.
- `cademi users products progress adjustments list` and `get`: `type` lists the five values the API returns (`reset`, `exam_result_deleted`, `exam_attempt_reopened`, `score_adjusted`, `score_deleted`) and is extensible, and `result` documents the keys of each type. In score adjustments, `result.score_id` and `result.product_id` are now public IDs (`sco_`, `prd_`) instead of internal numbers.
- Descriptions: a score balance is also corrected by deleting a manual entry (`cademi users scores delete`), the `score.deleted` event example has a null `exam_id` (manual deletion), and the comments list no longer mentions `include_replies`.
- `scores.delete` reports `introduced_in` 3.12.0 in the permission catalog, which moves to `catalog_version` 3.12.0.
- Version examples in the guide point to 0.2.6.

## cademi-cli 0.1.6 (2026-10-06)

- References follow the `cademi` CLI 0.2.5 (API 3.12.0): new `cademi users scores delete <user_id> <score_id>`, which deletes a manual point entry with a required `reason` and the `scores.delete` permission (automatic points return `422 score_not_removable`).
- The `score.removed` event is now `score.deleted`: pass `score.deleted` to `cademi listen --events`.
- The comments list also returns replies, and the `sandbox resets create` description cites the right scenario runs path.
- Version examples in the guide point to 0.2.5.
- This version also carries the changes of the `cademi` CLI 0.2.4 (API 3.11.0), since the skill was not published in between: `cademi support tickets list --queue` (`open`, `answered`, `closed`); `cademi support departments delete --unlink-tickets` (without it, a department with tickets is refused with `state_conflict`); `admin_id` in `support departments create` and `update`; `text` is optional in `support tickets replies create` (`file_ids` alone is enough); `products` is optional in `sales deliveries create`; `sales deliveries update` accepts `deleted` as an alias of `status`.
- Deliveries are live: `sales deliveries products update` and `sales deliveries rules update` apply to existing enrollments. A student reply in `support comments` can be deleted on its own, and `duration: null` in `users products update` inherits the delivery duration again.
- `admin_two_factor_available` in `settings authentication update` is deprecated, and the exit codes table notes that for a group in `permissions_any_of`, any one permission is enough.

## cademi-cli 0.1.4 (2026-10-06)

- References follow the `cademi` CLI 0.2.3 (API 3.10.1): `files exports create` and the three `processing-attempts create` commands (sales events, users imports, automations diamonds memberships) accept `--wait` and `--wait-timeout`, and required query flags (`account usage --period`, `reports activity list --from/--to`, `reports lessons list --product-id`, `reports support get --from/--to`) are marked `(required)`.
- The guide checks item-level failures with `cademi operations items <operation_id>`.
- Corrected command descriptions: `auth logout` points to `cademi integrations credentials update`, and `config apply` documents `--plan` as a file.
- Version examples in the guide point to 0.2.3.

## cademi-cli 0.1.3 (2026-10-06)

- References follow the `cademi` CLI 0.2.2 (API 3.10.1): updated descriptions of account usage, where `storage.used_bytes` is recalculated daily, and of uploads, with the 50 GiB storage quota and the attachment limit set by the account.
- Version examples in the guide point to 0.2.2.

## cademi-cli 0.1.2 (2026-10-05)

- The installation section explains `cademi skills install`, which keeps this skill at the version of the installed CLI, and `cademi mcp install <client>`, with a note on when to prefer the MCP over the CLI.

## cademi-cli 0.1.1 (2026-10-05)

- References follow the `cademi` CLI 0.2.1 (API 3.10.0): new `cademi mcp` (connect AI assistants to the Cademí MCP server) and `cademi skills` (install this skill) commands.
- `users` and `settings` references split by theme: `users-learning`, `users-profile`, `settings-platform`, `settings-access` and `settings-definitions`.

## cademi-cli 0.1.0 (2026-10-04)

- First release: agent guidance for the `cademi` CLI 0.2.0 (API 3.10.0) and one reference file per command domain.
