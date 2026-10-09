<!-- published at /help/admin/permissions.html on the Drive host -->

# Sharing and Permissions

## Scopes and owners

Every project lives in a **scope** — either a user's personal scope or a **group** scope. The scope is the "Owner" column on the Projects page. When you create a project you choose the scope: your own, or any group you can write to.

If a project lives in *your* scope, you have full (ADMIN) rights to it unconditionally.

## Permission levels

Access to a project is granted at one of these levels; higher levels include the lower ones:

| Level | Grants |
|---|---|
| **READ** | Open the project and view everything. |
| **WRITE** | Edit setup, upload data, run jobs, delete the project. |
| **EXECUTE** | Currently the same as WRITE. Members of a company's group are added at this level. |
| **ADMIN** | Everything, plus manage the project's member list and permissions. |

Your effective permission on a project is the *strongest* of: a direct grant on the project, a grant through group membership, and ownership (project in your own scope).

## Project members

On the Projects page, the **Project members** action on a row opens the access-control dialog (available to project ADMINs). Press **+** to add a row, type or pick the user — the suggestions are the members of your company's group; for anyone else, type the exact user name — choose their level (new rows start at READ), and save. Your own row is locked. The table shows direct grants only, not access that comes through a group.

## Groups

Groups (sidebar → **Groups**) share many projects with a team at once: projects created in a group's scope are accessible to the group's members, each at the level you give them in the group.

- **Add Group** — any user can create a group and becomes its administrator.
- **Edit** (group admins) — add or remove members and set each member's level over the group's projects.
- **Delete** — available to the group admin, and only once the group owns no projects.

Use a group scope for team-owned projects, and direct project membership for one-off sharing across team lines.

## API access

Automation and integrations authenticate with a token obtained from **Account → Get API Token**. A token carries *your* identity: anything using it can do exactly what you can do, on exactly the projects you can access, and nothing more. You hold one token at a time; it does not expire, and generating a new one revokes the old.

The [geosteering agent](../guide/agent.md) for Claude, ChatGPT and other hosts does not use this token: connecting it signs you in to Drive through your browser.
