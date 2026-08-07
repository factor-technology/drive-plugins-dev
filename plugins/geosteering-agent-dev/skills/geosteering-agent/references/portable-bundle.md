# Packaging Drive-MCP + Skill as a Portable Bundle

When a user asks to "bundle the geosteering agent skill for another
computer" (or any phrasing of: install on a fresh Hermes box, ship to
a customer machine, take it on the plane), ship **two tarballs**:

1. `geosteering-agent-skill.tar.gz` — the skill directory itself.
2. `drive-mcp-bundle.tar.gz` — the MCP tools the skill depends on,
   plus a wrapper script, an INSTALL.md, and a `secrets/` placeholder.

The skill tarball is useless without the MCP tools (the skill calls
`mcp_drive_*` tools that won't exist on the target machine). The MCP
bundle is useful without the skill but much weaker — the skill carries
the behavioral spec the agent should follow. Always offer both.

## Skill tarball

The skill lives at `~/.hermes/skills/geosteering-agent/`. It's pure
markdown — SKILL.md plus four reference docs. No env, no scripts, no
credentials.

    cd ~/.hermes/skills
    /opt/local/bin/bsdtar --no-mac-metadata --no-xattrs \
        -czvf /tmp/geosteering-agent-skill.tar.gz geosteering-agent

Target install: `tar -xzvf geosteering-agent-skill.tar.gz -C ~/.hermes/skills/`
and the skill auto-appears on the next message (no Hermes restart).

## Drive-MCP bundle

drive-mcp is **proprietary** — NOT on public npm. `npm install -g
drive-mcp` will 404. Build it from source — one command assembles the
whole bundle (esbuild the server, wrapper + secrets placeholder + config
snippet + INSTALL.md + the `.tgz`, then tar):

    yarn workspace agent build:skill        # the skill rides inside the .tgz
    yarn workspace agent build:mcp-bundle

That writes `agent/skill/dist/drive-mcp-bundle.tar.gz`,
already containing:

    drive-mcp-bundle/
      bin/drive-mcp-wrapper            # reads DRIVE_TOKEN from a secrets file
      secrets/drive_mcp_token.example  # placeholder ('<paste-your-JWT-here>')
      config-snippet/config.yaml.snippet
      drive-mcp-X.Y.Z.tgz              # the self-contained npm package
      INSTALL.md                       # rendered with the version

Ship that tarball as-is.

## Wrapper script pattern

Never put the JWT directly in `~/.hermes/config.yaml`. Use a wrapper
that reads from a secrets file:

    #!/bin/bash
    # ~/.hermes/bin/drive-mcp-wrapper
    set -euo pipefail
    TOKEN_FILE="$HOME/.hermes/secrets/drive_mcp_token"
    if [[ ! -r "$TOKEN_FILE" ]]; then
      echo "drive-mcp-wrapper: cannot read $TOKEN_FILE" >&2
      exit 1
    fi
    export DRIVE_TOKEN="$(cat "$TOKEN_FILE")"
    export DRIVE_BASE_URL="${DRIVE_BASE_URL:-https://drive.factor.technology}"  # prod default; set DRIVE_BASE_URL=https://drive-app-dev.factor.technology for dev
    exec drive-mcp "$@"

Config entry on the target (paths differ macOS vs Linux):

    mcp_servers:
      drive:
        command: /home/<user>/.hermes/bin/drive-mcp-wrapper   # or /Users/<user>/...
        timeout: 120

## INSTALL.md template

Always include these sections, in this order:

1. **Prereqs** — Node ≥20, npm, Hermes.
2. **Install drive-mcp from the bundled tarball** — `npm install -g
   ./drive-mcp-X.Y.Z.tgz`. Call out explicitly that it's NOT on public
   npm.
3. **Wrapper + token** — copy wrapper to `~/.hermes/bin/`, drop JWT
   into `~/.hermes/secrets/drive_mcp_token`, chmod 600.
4. **Wire into Hermes config** — mcp_servers entry. Show both macOS
   `/Users/...` and Linux `/home/...` paths.
5. **Restart Hermes; smoke test.**
6. **Backend URL override** — `DRIVE_BASE_URL=...` env var to switch
   dev/prod.
7. **Token rotation** — overwrite secrets file, restart Hermes.
8. **Updating drive-mcp** — re-run `npm pack` on the source machine,
   ship new .tgz, `npm install -g ./drive-mcp-X.Y.Z.tgz` on target.
9. **Skill bundle (geosteering-agent)** — full install/verify/update/
   remove section, NOT a one-liner. The user actually noticed when this
   was light.

## Tar gotchas (macOS → Linux)

macOS's `/usr/bin/tar` and MacPorts' default tar (`/opt/local/bin/bsdtar`)
both embed AppleDouble files (`._foo`), `.DS_Store`, and extended
attributes. On extraction on Linux, GNU tar prints a wall of
`Ignoring unknown extended header keyword 'LIBARCHIVE.xattr.…'`
warnings. Not fatal, but ugly and alarms the user.

Suppress with:

    COPYFILE_DISABLE=1 /opt/local/bin/bsdtar \
        --no-mac-metadata --no-xattrs -czvf out.tar.gz dir/

MacPorts does NOT install GNU tar by default — only `bsdtar`,
`gpgtar`, and Perl's `ptar`. If you need true GNU tar (for
`--owner=0 --group=0` and similar), suggest `sudo port install
gnutar` first; it installs as `/opt/local/bin/gnutar`.

## YAML indentation gotcha

Users often mis-indent the `mcp_servers:` block when adding it by
hand. Common mistake:

    mcp_servers:
        drive:
        command: /home/.../drive-mcp-wrapper   # WRONG: sibling of `drive:`
        timeout: 120

Correct (2-space or 4-space — must be consistent with file's existing
style, but children must indent deeper than parents):

    mcp_servers:
        drive:
            command: /home/.../drive-mcp-wrapper
            timeout: 120

Sanity check after editing:

    python3 -c "import yaml; print(yaml.safe_load(open('~/.hermes/config.yaml'.replace('~', __import__('os').path.expanduser('~'))))['mcp_servers']['drive'])"

Should print a dict with both `command` and `timeout`.

## Verification on the target machine

After install, ask Hermes:

    "list my registered WITSML servers"

That triggers `mcp_drive_list_witsml_servers` — confirms drive-mcp is
wired AND the JWT works AND `DRIVE_BASE_URL` is reachable. A clean
result (even an empty list) means the whole stack is healthy.

For the skill, ask:

    "load the geosteering-agent skill"

Should return SKILL.md content. If Hermes says it doesn't exist,
the tarball extracted to the wrong place — `ls
~/.hermes/skills/geosteering-agent/SKILL.md` is the quick check.
