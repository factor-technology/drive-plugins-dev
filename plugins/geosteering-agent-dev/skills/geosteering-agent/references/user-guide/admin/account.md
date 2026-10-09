<!-- published at /help/admin/account.html on the Drive host -->

# Account and Notifications

Your account page (sidebar → your user name) collects security, integration, and notification settings.

## Security

- **Change Password** — update your password.
- **Set up Passkey** — register a passkey for passwordless sign-in.

## Account links

- **Get API Token** — issue a token for API access. The token acts as you, with your permissions; you hold one at a time, it does not expire, and generating a new one revokes the old. See [Sharing and Permissions](./permissions.md#api-access). The **API** link on the Projects sidebar opens the REST API reference.
- **Manage WITSML Servers** — your registered WITSML servers; see [WITSML Servers and Polling](./witsml.md).
- **Job Run History** — your job runs over a date range (the last three days by default): project, scope, MD reached, start time, size, kind (extension or rerun), interval, and duration, with a CSV download that adds each run's billing status. Useful when diagnosing a failed or slow run.

## Notifications

Drive can alert you when a run flags the wellbore [out of zone or off the target line](../guide/setup/run-job.md#notifications) — the two opt-ins on each project's Run Job step. There is no job-finished notification. The check happens at the end of each run's forward phase, and alerts go out through three channels:

| Channel | Settings |
|---|---|
| **Pushover** | Your Pushover user key, plus a priority: None (off), Normal, High, or Critical. |
| **SMS** | Toggle plus a ten-digit North American phone number. |
| **Email** | Toggle; sends to your account email address. |

## Email data ingestion

Distinct from notifications: projects can *receive data* by email. That per-project address, its allowed senders, and the curve to extract are configured in the project's [Active Well step](../guide/setup/active-well.md#email), not here.
