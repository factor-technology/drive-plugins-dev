# Drive HTTP Fallback Patterns

> **Local runtime only.** Everything here runs through `drive_http_request`,
> which exists only in the locally-installed drive-mcp server. The remote
> connector doesn't carry it — a remote client never holds a Drive token, so
> there is no raw-HTTP path to fall back to. If `read_local_las` is missing
> from your tool list you are on the remote connector: this file does not
> apply, and an un-wrapped endpoint is a "we don't have a tool for that yet"
> answer to the user, not a workaround to improvise.

Most `drive-mcp` tools are project-scoped — they require a `{scope, name}`
project_id you already know. Plain listing and name search are the exception:
the `list_projects` tool handles them (spans all scopes by default, with
optional `scope` / `name_contains` filters), so reach for it first for "what
projects do I have under scope X" or "find all projects whose name contains Y".

For everything the curated tools still don't cover — a one-off endpoint, an
image body, **bulk MPE/result extraction across many projects** — use the
**`drive_http_request` tool**, NOT `curl` and NOT a browser.

## Required: use `drive_http_request`, not curl / Chrome

`drive_http_request` is the raw HTTP escape hatch over the Drive v2 API. It is
the required way to hit an un-wrapped endpoint:

- The **JWT is injected server-side** — it never enters your context. Do not
  read the token file, do not put `Authorization` headers in scripts, do not
  shell out to `curl -H "Authorization: Bearer ..."`. The whole point of the
  tool is to keep the credential out of the model.
- It is **structurally single-host**: you pass a host-relative `path` (starting
  with `/`), never a full URL. The host is fixed to the configured Drive
  backend (`DRIVE_BASE_URL`). Absolute or cross-host URLs are refused.
- **Methods:** GET / PUT / POST, plus DELETE *except* where a curated tool
  already owns the irreversible op (e.g. project delete is denied here — use
  `delete_project`).
- **Responses:** JSON / text come back inline, truncated to `max_chars`. Binary
  (images, octet-stream) and anything with `save_to_file=true` are written to a
  local temp file and only the path is returned — image bytes and large MPE
  blobs never transit the model.
- **Request bodies:** `body` for JSON; `body_file` (a local path) + a
  `content_type` for raw bytes, e.g. uploading an image to `/.../image/body`.

Examples:

```
# List projects in a scope
drive_http_request({path: "/api/v2/projects?scope=hw1"})

# Fetch an image to disk (returns {file_path, content_type, total_bytes})
drive_http_request({path: "/api/v2/hw1/projects/MyWell/image/body", accept: "image/png"})

# PUT a raw image body from a local file
drive_http_request({path: "/api/v2/hw1/projects/MyWell/image/body",
  method: "PUT", body_file: "~/Downloads/section.png", content_type: "image/png"})
```

`curl` / `urllib` / a headless browser are a **last resort** only if
`drive_http_request` is unavailable in the runtime — and then the auth and
User-Agent gotchas below apply. The base URL the tool uses defaults to prod
(`https://drive.factor.technology`); installs override `DRIVE_BASE_URL` to
`https://drive-app-dev.factor.technology` for the dev backend.

## Known endpoints (dev backend)

| Purpose | Endpoint | Notes |
|---|---|---|
| **Bulk full project objects** (triage / grouping) | `GET /api/v2/projects` (optionally `?scope=<scope>`) | Returns a JSON array of the **full** project objects (dip_type, md range, metadata, geo_target, etc.) for every project the caller can read — across all scopes by default. Use this *only* when you need rich fields across many projects in one call (e.g. group/triage by `dip_type` or `geo_target`). For plain "what projects do I have", name search, or recency, use the **`list_projects` tool** (a thin {name, scope, latestActivity, modifiedOn} digest); for one project's full detail use **`read_project`**. `scope` is an optional filter, rarely needed. |
| **OpenAPI spec** | `GET /download/openapi.json` | The full Drive v2 API contract for the configured backend — field meanings/types, endpoint params, request/response schemas. Public, unauthenticated. Reach for it to recover the semantics of a field or endpoint the curated tools don't wrap. It's large (~150 KB, ~45K tokens), so **fetch with `save_to_file: true`** and read bounded slices with `read_local_text` rather than inlining the whole document. Carries definitions only — NOT the UI↔stored tolerance conversions (those live in the behavioral spec). |
| Per-project everything | Use the `mcp_drive_*` tools — they encode `{scope, name}` into the right paths. | — |

Path-style variants that **DO NOT WORK** (return 404 HTML, easy to mistake for a
404 JSON):
- `/api/v2/<scope>/projects`
- `/api/drive/v1/<scope>/projects`
- `/api/v1/<scope>/projects`

The query-string form (`?scope=`) is the only listing variant the dev server
serves. If you find another scope-listing endpoint later, add it here.

## Reference recipe — bulk full-object triage

For plain listing, name search, or recency, use `list_projects` — not this. The
raw call earns its keep only when you need the **full** project objects in bulk,
which `list_projects` (a four-field digest) and `read_project` (one project at a
time) can't give you in a single call:

```
drive_http_request({path: "/api/v2/projects"})
# → JSON array of every accessible project as a FULL object. Group / triage on
#   dip_type, md_last, geo_target, metadata.created.on, etc. in your reasoning.
#   Append ?scope=<scope> only to narrow to one scope.
```

Don't dump the raw blobs into chat; project at the field level.

## If you must shell out — auth + User-Agent gotchas

These apply ONLY to the last-resort `curl`/`urllib` path (the tool handles auth
and UA for you, so you should rarely be here):

- Token file: `~/.hermes/secrets/drive_mcp_token` (a JWT — `sub=<username>`,
  `aud=/api/drive/v1`, `iss=Factor Technology`, **no `exp` claim**; rotation is
  what kills it, not expiry). Auth header:
  `Authorization: Bearer $(cat ~/.hermes/secrets/drive_mcp_token)`.
- The dev backend's gateway **403s Python's stdlib `urllib.request`** (UA
  `Python-urllib/3.x`). `curl` works, the MCP works, but hand-written
  `urllib.request.urlopen(...)` silently 403s on every request unless you
  override UA to e.g. `curl/8.0`. If `requests` is available, prefer it (its
  default UA passes). This is a server-side rule, not an environment quirk.

## MPE / manifest extraction (bulk results pull)

Pulling MPE rows across many projects (e.g. one CSV of `MD, formation_tvd` and
another of `MD, pmax_1` for the whole portfolio) requires the manifest →
MPE-blob chain. The `mcp_drive_read_latest_job_result` /
`mcp_drive_read_mpe_slice` tools work per-project but are noisy at scale; drive
the chain with `drive_http_request` instead.

Two-step recipe:

1. `drive_http_request({path: "/api/v2/<scope>/projects/<urlenc-name>/manifest-long"})`
   → returns `{"mpe": "run/<runID>/mpe/<key>.json", "marginals": [...],
   "job": ...}`. The `mpe` value is a relative ref split as
   `run/<runID>/mpe/<key>`.
2. `drive_http_request({path:
   "/api/v2/<scope>/projects/<urlenc-name>/job/run/<urlenc-runID>/mpe/<urlenc-key>",
   save_to_file: true})` → writes the full MPE blob to a local temp file and
   returns its path (these blobs are large — save_to_file keeps them out of
   context). The blob is keyed by zero-padded MD strings (e.g. `"010500"`),
   plus a `probs_pct` array at top level (skip that key when iterating). Each MD
   entry carries `formation_tvd` (MPE depth, TVDSS ft positive-up on the 0.5 ft
   grid) plus `pmax_1..pmax_9` and `pmean_1..pmean_9` (the nine alternatives —
   suffix is an explanation index, not a rank).

URL-encode each path segment separately (e.g. `encodeURIComponent`) because
project names contain spaces, dashes, and sometimes slashes. Don't encode the
whole path or you'll double-encode the slashes.

If `drive_http_request` returns `{"error": true, "status": 401, ...}` for
**every** call (e.g. `list_witsml_servers`, project reads also 401), the token
in `~/.hermes/secrets/drive_mcp_token` was rotated. The JWT itself may still
parse cleanly (no `exp`); the dev backend rotates its signing key or revokes the
`jti`, which is invisible to client-side inspection.

Remediation: ask the user to mint a fresh dev token and overwrite the file.
Avoid trailing newlines: `printf '%s' "$NEW_TOKEN" > ~/.hermes/secrets/drive_mcp_token`.

A 401 on ONE endpoint but 200 on others is a different problem (wrong scope,
not in the ugroup, etc.) — don't pin it on the token.

## When to add a real MCP tool vs keep using `drive_http_request`

`drive_http_request` is the catch-all for the long tail — but it is a fallback,
not a destination. Project listing was the first case where shelling out got
repetitive, so it is now the `list_projects` tool. Apply the same rule to any
future query: if you find yourself making the same `drive_http_request` call on
every cycle, promote it to a curated tool rather than leave it as a raw-HTTP
recipe here.
