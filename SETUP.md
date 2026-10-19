# Set up Patchy MCP for Codex CLI

Follow these instructions on the machine where Codex runs. Support is currently
Codex CLI on macOS with Node 22.22 or newer. The MCP add-on is a separate package.
Do not clone source, compile it, copy credentials, or manually edit Codex config.

## 1. Check prerequisites

- Check `node --version` and `npm --version`. If missing or older than Node 22.22,
  install the current Node LTS through the machine's existing version manager or
  the [official Node installer](https://nodejs.org/en/download). Recheck versions.
- Check `codex --version`. If missing, use the existing npm installer:
  `npm install --global @openai/codex@0.160.1`. If Codex needs its own login, follow
  its existing login instructions. Never replace someone else's Codex login.
- If `patchy` is missing or doctor below reports an old CLI, use Patchy's existing
  installer. Download https://cloud.patchyhq.com/install.mjs to a temporary file
  and run it with Node. It verifies and installs the public Patchy release.
  Keep the installed executables on PATH. Use a user-owned npm prefix if needed.
  This is MCP setup only; do not start the broader company-look/welcome-page flow.

## 2. Install the prebuilt add-on

Download both files, without GitHub login:

- https://github.com/vsrikar08/patchy-mcp/releases/download/v0.2.0/patchy-mcp-0.2.0.tgz
- https://github.com/vsrikar08/patchy-mcp/releases/download/v0.2.0/patchy-mcp-0.2.0.tgz.sha256

Verify the archive with the downloaded SHA-256 file. Then install:

```sh
npm install --global --ignore-scripts --no-audit --no-fund https://github.com/vsrikar08/patchy-mcp/releases/download/v0.2.0/patchy-mcp-0.2.0.tgz
patchy-mcp --version
patchy-mcp doctor
```

Expected MCP version: `0.2.0`. The tarball includes built JavaScript and all runtime
dependencies. No development tools or build scripts are needed.

Clear an old `PATCHY_API_TOKEN` override before setup. The add-on uses Patchy's
normal saved machine login, not a token in agent configuration.

## 3. Sign in only if needed

If doctor asks for sign-in, run:

```sh
patchy-mcp login --json
```

Show the person the returned `verificationUrl` and `userCode`. They open that URL
in their own browser, sign in to their existing/invited Patchy company, check the
code/account/company, name the machine and confirm. Do not open or confirm the
browser flow on their behalf. An invitation or approved beta access is required.

Run the returned `next` command with `--json` added. If it reports `pending`, resume
the same command after confirmation; proceed only when status is `logged_in`.
The existing CLI saves the credential privately. Run `patchy-mcp doctor` again and
check that it shows the intended account/company. Never display the secret-bearing
`patchy auth token` response or read the private credentials file.

## 4. Connect and verify

```sh
patchy-mcp setup codex
codex mcp get patchy
```

Setup verifies the saved login and uses Codex's own registration command. It stores
`patchy-mcp` as the executable name and the Patchy URL; it stores no token or source
checkout path. It replaces only an existing `patchy` MCP registration.

Start a fresh Codex session so it loads the new server. Ask it:

> Use Patchy's list_patches tool to list the patches I can access.

Success means the actual `list_patches` MCP call returns Cloud's patch inventory
without a tool error. An empty inventory is valid. Do not substitute a shell/CLI
listing for this final verification. If the host requires tool approval, let the
person approve the read. Report the account/company and the tool result.

## Sign in again or disconnect

A 401 means repeat the same `patchy-mcp login --json` handoff, complete its `next`,
then retry the tool. The running MCP re-reads the saved credential on every call.
Signing in as another account changes the identity used by both MCP and the CLI.
A missing/old CLI needs the existing Patchy installer; a network failure or a
forbidden patch is not a reason to invent an empty result or automatically log out.

`patchy-mcp remove codex` disconnects this add-on and keeps the shared Patchy login.
`patchy-mcp logout` signs this machine out of Patchy through the existing CLI.
`npm uninstall --global patchy-mcp` removes the executable.

Other clients and hosted MCP are outside this release. See the
[official Codex MCP documentation](https://developers.openai.com/codex/mcp/) for
Codex's registration and restart behavior.
