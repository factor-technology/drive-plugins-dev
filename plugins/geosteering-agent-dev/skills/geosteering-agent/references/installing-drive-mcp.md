# Installing the drive-mcp Tool Bundle

The geosteering agent needs the Drive tool catalog (~87 tools: UI parity for
project setup, tuning, alignment, runs, WITSML). There are **three ways to get
it**:

- **Local install** (this document, §1–§4) — the `drive-mcp` server runs on the
  user's own machine from a **single self-contained npm tarball**: no monorepo
  clone, no source checkout, no compilation. It is the only option that can
  read the user's files, and it carries the raw HTTP escape hatch.
- **Remote connector** (§5) — nothing to install and no token to paste; the
  user authorizes Factor Drive in their host's connector settings. Serves 82 of
  the 87 tools (see §5 for what's missing and what replaces it).
- **Plugin** (§6) — the remote connector and this skill in one installable
  file, for hosts that take plugins. Same tools and caveats as §5, minus the
  manual connector setup.

Both serve the same catalog from the same source, so the behavioral spec is
identical; the differences are listed in §5.

## 1. Install the tarball

You'll be given a `drive-mcp-X.Y.Z.tgz` file (email / SFTP / S3 — the
package is **proprietary; NOT on public npm**. `npm install -g drive-mcp`
will 404. The maintainer ships the .tgz directly).

**If you're working in the drive-app monorepo itself** (developer /
maintainer install, not a customer install), build it from source with
`yarn workspace agent build:mcp-bundle`. The freshly-built
tarball lands at
`~/drive/drive-app/agent/skill/dist/drive-mcp-bundle/drive-mcp-X.Y.Z.tgz`
(the npm-pack source — `drive-mcp.js`, `version.json`, `meta.json`, the bundled
`skill/` — lives next door in `../drive-mcp-pkg/`). Use
that tarball directly — no need to fetch externally.

```bash
# Recommended: global install
npm install -g /path/to/drive-mcp-X.Y.Z.tgz
which drive-mcp                       # confirms it's on PATH

# Or: project-local
mkdir ~/drive-mcp && cd ~/drive-mcp
npm init -y
npm install /path/to/drive-mcp-X.Y.Z.tgz
# binary at ./node_modules/.bin/drive-mcp
```

Requires Node.js ≥ 20.

The tarball bundles every JS dep inline. The only thing the customer's
machine needs is Node.

## 2. Get a Drive JWT

The MCP server acts as a single Drive user. Get a long-lived JWT for
that user:

```js
// from a Drive shell
const {tokenForUsername} = require('./server/utils/jwt/tokens.js')
const token = await tokenForUsername({username: 'you@factor.technology'})
console.log(token)
```

**Node ≥ 22 caveat — `require()` doesn't work.** `server/utils/jwt/tokens.js`
is an ESM module with top-level await; `require()`-ing it from a `node -e`
script fails with `ERR_REQUIRE_ASYNC_MODULE`. Use dynamic `import()` inside
a module-mode eval instead:

```bash
cd ~/drive/drive-app
node --input-type=module -e "
  const {tokenForUsername} = await import('./server/utils/jwt/tokens.js');
  console.log(await tokenForUsername({username: 'you@factor.technology'}));
"
```

This requires a reachable Drive Postgres — the script reads `jwts` from
the DB. If you see `error: role "drive" does not exist` (or any
connection error), the local DB isn't configured. Either (a) point at
the dev/prod DB via the standard `DB_*` / `DATABASE_URL` env vars that
`server/datamodel/db.js` consumes (may require a tunnel/bastion if the
DB lives inside the AMI security group), or (b) skip the script and
reuse an existing token captured elsewhere.

The token inherits that user's project permissions. Drive's middleware
enforces per-project access on every call, so the MCP server cannot do
anything the underlying user couldn't.

## 3. Wire it into Hermes

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  drive:
    command: "drive-mcp"           # global install
    env:
      DRIVE_TOKEN: "eyJhbGciOiJIUzI1NiJ9..."
      DRIVE_BASE_URL: "https://drive.factor.technology"
    timeout: 120
```

If installed project-local instead:

```yaml
mcp_servers:
  drive:
    command: "node"
    args: ["/Users/you/drive-mcp/node_modules/.bin/drive-mcp"]
    env: {DRIVE_TOKEN: "...", DRIVE_BASE_URL: "https://..."}
```

Restart Hermes. Tools appear as `mcp_drive_<tool_name>`. Pair with the
`geosteering-agent` skill so the model gets the behavioral spec
alongside the tools.

Verify with:

```
> list the mcp_drive_* tools you have
```

You should see the full local catalog: `mcp_drive_read_latest_job_result`,
`mcp_drive_align_logs`, `mcp_drive_update_param_block`,
`mcp_drive_drive_http_request`, etc.

## 4. Wire it into Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "drive": {
      "command": "drive-mcp",
      "env": {
        "DRIVE_TOKEN": "eyJhbGciOiJIUzI1NiJ9...",
        "DRIVE_BASE_URL": "https://drive.factor.technology"
      }
    }
  }
}
```

Restart Claude Desktop. Tools appear under the "drive" server in the
hammer/MCP icon.

## 5. Or: connect the remote connector (no install)

The Drive server serves the same tool catalog over HTTP as an MCP connector.
Nothing is installed and **no JWT is generated or pasted** — the user
authorizes with their ordinary Drive login.

Setup, in the host's connector/integration settings (claude.ai → Settings →
Connectors, Claude Desktop → Settings → Connectors, or the equivalent):

1. Add a custom connector with the URL `https://<drive-host>/mcp` — the same
   host the user opens Drive on (e.g.
   `https://drive-app-dev.factor.technology/mcp`). Ask them for it; never
   guess prod vs dev.
2. The host discovers the rest on its own: the unauthenticated first call
   returns a challenge pointing at the site's protected-resource metadata
   (RFC 9728), which names the authorization server. The host registers
   itself, opens a browser for the user to sign in to Drive and approve the
   permissions, and stores the resulting token. There is no client id, secret,
   or callback URL for the user to configure.
3. Tools appear under the connector's name. Permissions can be re-approved
   later by reconnecting — which is the fix when a call is refused for lack of
   a permission scope.

Differences from the local install (the agent detects which it is on by
whether `read_local_las` is in the tool list):

| | Local | Remote connector |
|---|---|---|
| Credential | operator's Drive JWT in the server env | per-user OAuth token, minted by Drive |
| Users per install | one | many (multi-tenant) |
| Reads the user's files | yes (`read_local_las`, `read_local_text`, `file_path`) | no |
| Large LAS ingest | `file_path` | `create_upload` + a send from code execution, then `upload_id` |
| `drive_http_request` | yes | no |
| Long-runners | `align_logs` / `refresh_witsml_server` (sync) or `start_*` | `start_*` + `get_operation_status` only |

Two remote-only notes worth knowing:

- **Code-execution egress.** The large-file send runs from the host's code
  execution sandbox. Locked-down organizations block outbound traffic there
  (MCP traffic itself is exempt), which makes the send fail with a network
  error; the fix is an admin allowlisting the Drive domain for code execution.
- **Permission scopes.** Tools are grouped into read / write / run / admin
  permissions. A call outside the granted set is refused with a scope error
  naming what it needs — self-serviceable by reconnecting and approving the
  fuller set, not a support case.

## 6. Or: install the plugin (connector + skill in one file)

`geosteering-agent-<version>.plugin` packages this skill together with the
remote connector's server entry, so §5's setup is already done: the user
installs one file and approves Factor Drive on the first tool call. Hosts that
take plugins (Cowork, Claude Code) accept the file directly.

It is built per backend, because the URL is baked in — `geosteering-agent-<v>`
is production, `geosteering-agent-dev-<v>` is dev, and the dev one carries a
distinct plugin name so both can be installed at once. Hand a customer the
production file. The tools, scope prompts, and local-only gaps are §5's
exactly. Build both with `yarn workspace agent build:plugin`.

## Upgrading

drive-mcp **tells you when an upgrade is available.** On every startup
it fetches a small `version.json` from the hosted URL and, if a newer
release is out, logs a one-line warning to its stderr (which Hermes /
Claude Desktop surface in their MCP server logs):

```
[drive-mcp] update available: 0.1.1 → 0.2.0 — fixed alignment edge case (install: npm i -g https://drive.factor.technology/drive-mcp/drive-mcp-0.2.0.tgz)
```

To upgrade, run the suggested command and restart your agent runtime:

```bash
npm install -g https://drive.factor.technology/drive-mcp/latest.tgz
# or, to pin a specific version:
npm install -g https://drive.factor.technology/drive-mcp/drive-mcp-0.2.0.tgz
```

No config changes needed. Rollback works the same way — install an
older versioned tarball.

**Disable the check** (e.g. for offline environments) with
`DRIVE_MCP_NO_VERSION_CHECK=1` in the MCP server env block.

**Point at an internal mirror** with `DRIVE_MCP_VERSION_URL=https://…`
in the env block (must serve the same `{version, tarball_url, notes}`
JSON shape).

The check is best-effort: a missing or unreachable version.json never
breaks startup, just skips the warning that cycle.

When a private npm registry is wired up later, the same workflow
becomes `npm install -g drive-mcp@latest` after a one-time `.npmrc`
setup; the runtime notifier keeps working unchanged.

## What this bundle includes

**Full UI parity for setup + tuning:**

- Read/diagnostic: job result, marginal, MPE slice, job params, user
  prior, structure, fit params, pilot list, interpretation list, WITSML
  server/poller/crawl.
- Project lifecycle: `create_project`, `delete_project`.
- Pilot wells / active well: pilot create + log upload, markers, top
  of target, active trajectory + plan + log, email config, ignore
  ranges, data-source mode.
- Dip & azimuth (Step 3), Align Logs (Step 4), Job Parameters (Step 5),
  Run Configuration (Step 6).
- WITSML server registration + per-project pollers + sync toggle.
- `trigger_job_rerun`.
- Async long-runners: `start_align_logs`, `start_refresh_witsml_server`,
  `get_operation_status` — the timeout-safe front ends, on both transports.
- Inline LAS parsing: `read_las` (content in, channels/samples out) and
  `create_upload` (the staged large-file send), on both transports.

**Local-only (absent from the remote connector):**

- Local files: `read_local_las`, `read_local_text`. The LAS upload tools
  (`upload_pilot_log`, `upload_active_log`) take a local `file_path`, so a
  LAS file on the operator's machine is read and parsed inside the MCP
  server — the samples never transit the model. Served remotely, `file_path`
  would name the Drive server's own disk, so the tools refuse it there.
- Raw HTTP escape hatch: `drive_http_request` — single-host (the configured
  backend), JWT injected server-side, for endpoints no curated tool wraps
  yet (image bodies, bulk MPE pulls). Binary/large responses are written to a
  local temp file rather than returned into the model.
- Synchronous `align_logs` / `refresh_witsml_server` — their multi-minute work
  can't answer inside a hosted request, so the remote connector serves the
  `start_*` pair instead.

## Troubleshooting

- **"DRIVE_TOKEN is required"** — env block missing in the MCP server
  config; the server exits before connecting.
- **"fetch failed" on every tool** — `DRIVE_BASE_URL` unreachable.
  Check the URL and your network/VPN.
- **401 on every tool** — JWT expired or revoked. Generate a new one.
- **Tools not appearing in Hermes** — `mcp` Python package isn't
  installed (`pip install mcp`), or the config lives under `mcp` /
  `servers` instead of `mcp_servers`. On a fresh Hermes install the
  `mcp_servers:` key may not exist at all in `~/.hermes/config.yaml` —
  add it at the top level (sibling of `model:`, `providers:`, etc.);
  don't nest it under another section.
- **Subset of tools 4xx** — the user's project permissions don't
  cover the operation. The tool returns `{"error": true, "status":
  403, ...}` rather than crash.
- **Remote connector: every call unauthorized** — the authorization expired or
  was revoked. Reconnect in the host's connector settings; there is no token
  file to edit.

## Verification: smoke-test without Drive

You can confirm the server starts and lists tools without a
reachable Drive:

```bash
DRIVE_TOKEN=fake DRIVE_BASE_URL=http://127.0.0.1:1 drive-mcp
# Reads JSON-RPC on stdin. Type Ctrl-D to exit. To exercise it
# programmatically, use any MCP client (e.g. @modelcontextprotocol/sdk).
```

The server should start cleanly. Tool calls will return
`{"error": true, "note": "fetch failed"}` because Drive isn't
reachable — that's expected and proves the tool dispatch path works.
