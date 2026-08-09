# Connecting the Drive Tool Catalog

The geosteering agent needs the Drive tool catalog (~82 tools: UI parity for
project setup, tuning, alignment, runs, WITSML). It is served over HTTP by the
Drive server itself — there is **nothing to install on the user's machine and
no token to paste**. There are **two ways to get it**:

- **The plugin** (§1) — the connector and this skill in one installable file.
  The setup below is already done; the user installs one file and approves
  Factor Drive on the first tool call.
- **The connector on its own** (§2) — the user adds a custom connector in their
  host's settings. Same catalog, same authorization flow; use this when the
  host takes connectors but not plugins, or when the skill arrived separately.

## 1. Install the plugin (connector + skill in one file)

`geosteering-agent-<version>.plugin` packages this skill together with the
connector's server entry. Hosts that take plugins (Cowork, Claude Code) accept
the file directly.

It is built per backend, because the URL is baked in — `geosteering-agent-<v>`
is production, `geosteering-agent-dev-<v>` is dev, and the dev one carries a
distinct plugin name so both can be installed at once. Hand a customer the
production file. Build both with `yarn workspace agent build:plugin`.

A hand-delivered `.plugin` never refreshes itself. Installing from the
**marketplace** repo instead gives the install an update channel — see
`agent/skill/README.md` for publishing, and note that auto-update is still off
by default and the user has to turn it on.

## 2. Or: add the connector by hand

Every host that speaks MCP over HTTP can reach it this way — Claude Desktop,
Hermes, Cowork, ChatGPT. In the host's connector/integration settings
(claude.ai → Settings → Connectors, Claude Desktop → Settings → Connectors, or
the equivalent):

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

The authorization is the user's own Drive login, so the tools inherit exactly
that user's project permissions. Drive's middleware enforces per-project access
on every call; the connector cannot do anything the underlying user couldn't.

Two notes worth knowing:

- **Code-execution egress.** The large-file send runs from the host's code
  execution sandbox. Locked-down organizations block outbound traffic there
  (MCP traffic itself is exempt), which makes the send fail with a network
  error; the fix is an admin allowlisting the Drive domain for code execution.
- **Permission scopes.** Tools are grouped into read / write / run / admin
  permissions. A call outside the granted set is refused with a scope error
  naming what it needs — self-serviceable by reconnecting and approving the
  fuller set, not a support case.

## What the catalog includes

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
  `get_operation_status` — the timeout-safe front ends.
- Inline LAS parsing: `read_las` (content in, channels/samples out) and
  `create_upload` (the staged large-file send).

**Deliberately absent** — the catalog runs inside the Drive server, shared by
every user, so it carries no filesystem readers and no raw HTTP escape hatch:

- No local-file tools, and no tool takes a path: a path would name the Drive
  server's own disk. A large LAS comes in through `create_upload` plus a send
  from code execution, then `upload_id`; a survey's rows are parsed by the
  agent and passed inline.
- No blocking long-runners. Alignment and WITSML re-crawls take minutes,
  which no hosted request can wait out, so they are started with
  `start_align_logs` / `start_refresh_witsml_server` and polled with
  `get_operation_status`.

## Troubleshooting

- **Every call unauthorized** — the authorization expired or was revoked.
  Reconnect in the host's connector settings; there is no token file to edit.
- **A call refused naming a permission** — the granted scopes don't cover it.
  Reconnect and approve the fuller set.
- **Subset of tools 4xx** — the user's project permissions don't cover the
  operation. The tool returns `{"error": true, "status": 403, ...}` rather
  than crash.
- **No tools at all** — no connector is configured (or the plugin didn't
  install). Point the user at §1 or §2; do not guess the Drive host for them.
- **The large-file send fails with a network error** — code-execution egress is
  blocked; an admin has to allowlist the Drive domain.
