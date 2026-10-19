# Patchy MCP

A small, standalone MCP add-on for discovering published Patchy Cloud patches. It is not part of the `patchy-cloud` workspace and does not import its packages. It exposes four read-only tools: `list_patches`, `get_patch` (metadata), `read_page` (the current tier 0 document), and `check_launch_queue` (one published Tool query).

## Install and connect

Download the versioned `patchy-mcp-0.2.2.tgz` release and its SHA-256 file, then give
[SETUP.md](SETUP.md) to your agent. It installs the prebuilt executable, reuses
Patchy's existing machine login, and registers it with Codex CLI:

```sh
npm install --global --ignore-scripts https://github.com/vsrikar08/patchy-mcp/releases/download/v0.2.2/patchy-mcp-0.2.2.tgz
patchy-mcp --version
patchy-mcp doctor
# If needed: patchy-mcp login --json; relay URL/code and complete returned next.
patchy-mcp setup codex
```

Restart Codex or start a new session. No token is placed in its config and no
absolute executable path is required. Only the person confirms their device login
in the browser. The public Patchy CLI must be on PATH. Doctor uses `patchy whoami --json`;
patch discovery uses the existing `patchy list --json` interfaces. These commands
own credential selection and authentication, so setup works with the published
CLI without a new credential exporter or a Cloud deployment.

The prebuilt tarball and agent instructions are published at
https://github.com/vsrikar08/patchy-mcp/releases/tag/v0.2.2.
Additional Page/query operations still require the prototype's corresponding
Cloud API and credential interface; onboarding verifies the existing discovery
operation `list_patches`.

`PATCHY_API_URL` defaults to `https://cloud.patchyhq.com`. For a custom instance,
use `--api-url` on doctor, login and setup. Advanced direct launches still accept
`PATCHY_API_TOKEN` as an explicit override. Otherwise discovery asks the public CLI to make each Cloud request with its current saved login.
A 401 gives “sign in again” steps; completing login repairs the running connection.

`patchy-mcp remove codex` disconnects MCP. `patchy-mcp logout` is shared machine
logout through the existing CLI. These are separate because another agent or CLI
may still use the machine's login. See SETUP.md for the full handoff and repair flow.

## Test

`npm run build && npm test` runs protocol, login-recovery and CLI setup tests against HTTP fixtures. `npm run release` creates the distributable and checksum under `artifacts/`; `npm run test:packed` checks the actual artifact from a clean installation. To also exercise a real running Cloud, publish a disposable company-only tier 0 Page there using the existing CLI, then run:

```sh
PATCHY_API_URL=http://127.0.0.1:<port> \
PATCHY_API_TOKEN=<machine-token> \
PATCHY_TEST_PAGE=<exact-page-name> \
PATCHY_TEST_TEXT=<expected-content> npm test
```

The opt-in Cloud test is read-only. It launches the actual MCP process, discovers the Page, reads its metadata, then verifies its content by both id and name. `PATCHY_TEST_TEXT` is optional. It never initializes or resets a database.

## One Tool operation

Ask: “Check Launch Queue and tell me what’s blocked.” The agent can select
`check_launch_queue` directly from its description. It takes `{}` and calls
`POST /api/patches/launch-queue/query` with `{ "handler": "queue.blocked", "args": {} }`.
Cloud executes the current published handler as the machine token’s user without an
open browser. The MCP result has this shape (also advertised as its output schema):

```json
{
  "patchId": "…",
  "versionId": "…",
  "blocked": [
    {
      "title": "Launch approval",
      "owner": "Mira Chen",
      "blocker": "Review pending"
    }
  ],
  "total": 1
}
```

`PATCHY_LAUNCH_QUEUE_PATCH` optionally pins a canonical patch id; otherwise the exact
name `launch-queue` is resolved in the token’s company. The handler name is fixed.
The companion Cloud query endpoint is required and has only been tested locally.
Cloud rejects foreign/disabled patches, off patches, invalid credentials, non-query
handlers, invalid inputs and excess request fields. The existing query executor
also blocks write capabilities. A handler refusal or malformed result becomes an
MCP tool error, never an invented empty queue.

The example in `examples/launch-queue` contains the patch config, one query and a
minimal read-only view. Initialize a tier 2 repo with the existing CLI, copy these
files over the starter, remove its `server/notes.ts`, run `patchy refresh`, then
publish with `patchy publish --share company`. The query reads the Tool’s actual
`items` table and follows pagination; it never returns example answers from MCP.
Local fixtures and demo database seeding are setup steps, not agent write tools.

Operation names, inputs and result shape are wired manually for this prototype.
Automatic discovery from published manifests, write operations and remote MCP
hosting remain later work.

To exercise the published query through stdio, set `PATCHY_TEST_LAUNCH_QUEUE=1`
alongside `PATCHY_API_URL` and `PATCHY_API_TOKEN`, then run `npm test`.
`PATCHY_TEST_BLOCKER` optionally asserts an exact expected blocker. This test reads
an already-published Tool; it never seeds or changes its table.
