Honor all existing project instructions and coding guidelines already active in this workspace. Do not replace them. Extend them only for the logging/CLI observability area using the instructions below.

Goal:
Improve the project’s logging and CLI output so it looks and behaves like a professional industry/enterprise server-side CLI application. Make only the important and necessary code changes. Do not over-engineer, do not rewrite unrelated code, and do not change business logic unless required to support logging/error presentation.

Context:
This project currently emits raw prints, dictionaries, partially formatted messages, and full Python tracebacks during normal CLI execution. The desired behavior is a clean operational story for the user, with detailed developer diagnostics available only in verbose/debug mode.

Please scan the entire project first, then implement a minimal, consistent logging improvement.

Logging principles to apply:
1. Separate audiences:
   - Default CLI output: human-readable progress, milestones, failures, and next actions.
   - Verbose/debug output: developer diagnostics, sanitized request/response details, stack traces.
   - Machine-readable logs, if already supported or easy to add: JSON/event-style logs.

2. Do not dump raw objects by default:
   - Replace raw `print(...)`, raw dictionaries, payload dumps, wallet paths, URLs, OCIDs, request bodies, and response bodies with meaningful log messages.
   - Keep sensitive or noisy details behind `--debug` only, and redact where appropriate.

3. Use professional log levels:
   - ERROR: operation failed or cannot continue.
   - WARNING: unexpected but recoverable condition.
   - INFO: meaningful lifecycle milestones.
   - DEBUG: developer details such as payload shape, HTTP details, stack traces, selected code paths.
   - Avoid INFO messages that say only “done” or expose implementation internals.

4. Each meaningful long-running operation should have:
   - started message
   - completed message with duration and useful counts
   - waiting/progress message if it may block for a noticeable amount of time
   - failed message with concise cause and next action

5. Prefer event-shaped messages:
   Examples:
   - `preflight.passed repository_version=... deployment_type=...`
   - `wallet.resolved adw=... services=5`
   - `datastore.filter.started project=...`
   - `datastore.filter.completed matched=... skipped=... duration=...`
   - `job.submit.started job=... object_type=... timeout=...`
   - `job.submit.failed status=400 reason="unsupported field isSynchronous"`

6. Default user-facing errors should be actionable:
   Instead of showing a raw traceback by default, show:
   - what failed
   - concise cause
   - relevant context
   - suggested next action
   - instruction to rerun with `--debug` for full traceback

7. Debug mode should still preserve developer power:
   - Full stack traces are acceptable only with `--debug`.
   - Sanitized HTTP method/path/status, payload keys, response snippets, retry attempts, and timings are useful.
   - Never log secrets, passwords, wallet contents, access tokens, private keys, or full connection strings.

8. stdout/stderr behavior:
   - Use stdout only for intentional command output.
   - Use stderr for logs, progress, warnings, and errors.
   - Keep CI friendliness in mind. Avoid spinners/colors unless already used or very lightweight.

Implementation expectations:
1. Inspect the project structure and identify:
   - CLI entrypoints
   - existing print statements
   - existing logging usage
   - exception handling boundaries
   - HTTP/client request handling
   - long-running operations or polling/waiting loops

2. Add or improve a central logging configuration if one does not already exist.
   - Prefer Python standard `logging`.
   - Do not add new dependencies unless the project already uses them or there is a strong reason.
   - Support existing verbosity flags if present.
   - If no flags exist, add minimal CLI flags such as `--verbose`, `--debug`, and optionally `--quiet` only if this fits the current CLI design.

3. Replace unprofessional output:
   - Remove accidental prints.
   - Replace raw object dumps with concise messages.
   - Fix broken formatting such as `wallet_location %s %s ...`.
   - Redact sensitive values.

4. Improve top-level exception handling:
   - For expected domain errors, show a clean error summary.
   - In debug mode, include traceback.
   - Preserve exit codes.
   - Do not swallow errors silently.

5. Add helper utilities only if useful:
   - `redact_sensitive(...)`
   - `format_duration(...)`
   - `log_step_started/completed/failed(...)`
   - or a small context manager for timed steps
   Keep these simple and idiomatic.

6. Add or update tests where appropriate:
   - default mode does not show traceback for handled errors
   - debug mode does show traceback
   - sensitive values are redacted
   - important lifecycle messages are emitted
   - existing behavior still works

7. Keep changes minimal:
   - Do not restructure the whole application.
   - Do not rename public APIs unnecessarily.
   - Do not change request payloads, schemas, or external behavior unless you find a clear bug directly related to logging/error reporting. If you find such a bug, explain it before changing.

Desired final behavior example:

Default CLI output should look closer to:

`INFO preflight.passed repository_version=05.02.02.25 dt_version=2026.01.21.03.00 deployment_type=ADB-S`
`INFO wallet.resolved adw=A31RLE17KFD74YW6 services=5`
`INFO datastore.filter.started`
`INFO datastore.filter.completed matched=12 skipped=4 duration=1.2s`
`INFO columns.resolve.completed tables=8 columns=143 duration=4.1s`
`INFO job.submit.started job=LOAD_CUSTOMERS object_type=MAPPING timeout=120s`
`ERROR job.submit.failed status=400 reason="server rejected request payload"`

Then a clean actionable error block:

`Job submission failed.`
`Cause: The server rejected the request field "isSynchronous". This API version expects "synchronous".`
`Endpoint: POST /dt-rest/v2/jobs/submit`
`Status: 400`
`Next action: update the submit-job payload mapping or choose the field based on Data Transforms API version.`
`Run again with --debug for full diagnostics.`

But debug mode may include sanitized payload details and traceback.

Deliverable:
After scanning, implement the smallest coherent set of changes needed to make logging professional and consistent. Then summarize:
- files changed
- logging behavior before vs after
- how to run default, verbose, and debug modes
- any sensitive fields now redacted
- any tests added or updated
