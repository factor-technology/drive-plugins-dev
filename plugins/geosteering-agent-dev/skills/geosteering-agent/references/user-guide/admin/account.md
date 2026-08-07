<!-- published at /help/admin/account.html on the Drive host -->

# Account and Notifications

Your account page (sidebar → your user name) collects security, integration, and notification settings.

## Security

- **Change Password** — update your password.
- **Set up Passkey** — register a passkey for passwordless sign-in.

## Account links

- **Get API Token** — issue a JWT for API access. The token acts as you, with your permissions; see [Sharing and Permissions](./permissions.md#api-access). The **API** link on the Projects sidebar opens the REST API reference.
- **Manage WITSML Servers** — your registered WITSML servers; see [WITSML Servers and Polling](./witsml.md).
- **Job Run History** — every job run under your account, with timing and status. Useful when diagnosing a failed or slow run.

## Notifications

Drive can alert you — for example when a job finishes or when a run flags the wellbore [out of zone or off the target line](../guide/setup/run-job.md#notifications) — through three channels:

| Channel | Settings |
|---|---|
| **Pushover** | Your Pushover user key, plus a message priority. |
| **SMS** | Toggle plus your phone number. |
| **Email** | Toggle; sends to your account email address. |

## Email data ingestion

Distinct from notifications: projects can *receive data* by email. That per-project address, its allowed senders, and the curve to extract are configured in the project's [Active Well step](../guide/setup/active-well.md#email), not here.
