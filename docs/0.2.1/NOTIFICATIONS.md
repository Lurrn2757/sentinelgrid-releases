# Set up alert delivery

Notifications run in the monitoring server. Existing standalone agents work with this release. The service begins sending when you enable a channel and restart the server. Use a test customer and chat or mailbox for your first activation.

## Connect a destination

1. In the extracted server package, make a local private copy of `notifications.example.json`. Keep only the destinations you want. Each entry requires a unique ID and an explicit tenant. Use the customer's exact login name. Use `"*"` only for an operations destination authorized to receive the entire fleet.
2. Configure the provider using the instructions below. Set `summary_minutes` to the desired interval, from 1 to 10080. Zero or omission enables incident and recovery delivery alone. The example Teams route sends an initial status and a fresh summary every five minutes, including healthy periods.
3. Install and validate the edited file. The following commands assume you saved it as `notifications.json` in the current directory and installed the server service.

```sh
sudo install -m 0600 -o sentinelgrid-server -g sentinelgrid-server notifications.json /etc/sentinelgrid-server/notifications.json
sudo -u sentinelgrid-server /usr/local/bin/sentinelgrid-server \
  -notifications /etc/sentinelgrid-server/notifications.json check-notifications
```

4. Set this variable in `/etc/sentinelgrid-server/config`, then run `sudo systemctl restart sentinelgrid-server`.

```ini
SG_NOTIFICATIONS=/etc/sentinelgrid-server/notifications.json
```

The check command validates the file locally. After activation, open the dashboard's **Notification delivery** section. For a first delivery check set the test route to a one-minute summary, confirm the message in the intended destination, then set the desired cadence and restart. Tenant dashboards show their scoped routes. Admin sees all routes. The service journal identifies a retry by channel ID. Secrets, email recipients, chat IDs and workflow URLs stay off the dashboard.

### Microsoft Teams group chat

Open **Workflows** in Teams and select **Send webhook alerts to a chat**. Choose the target group chat, sign in with an account that can post there, save the workflow, and copy its full webhook URL into `webhook`. Use the trigger authentication option **Anyone** for this integration. Access to the signed URL permits submissions, so store it as a credential. If your tenant disallows this trigger, an administrator must approve a suitable workflow before using this integration. The workflow posts the Adaptive Card to the chat you selected. Assign a co-owner for continuity when the workflow owner's account changes. [Microsoft Teams webhook setup](https://support.microsoft.com/en-gb/workflows/send-messages-in-teams-using-incoming-webhooks)

### Telegram

Create a bot with BotFather and set `bot_token`. For a private chat, the user must start the bot. For a group, add the bot to that group and allow it to send messages. Set `recipient` to the numeric chat ID as a quoted string. A channel username is also supported for a channel where the bot can post. Use separate route IDs for different chats. [Telegram Bot API](https://core.telegram.org/bots/api#sendmessage)

### Gmail and Google Workspace

Enable the Gmail API for your Google Cloud project, configure the OAuth consent screen and create an OAuth client. Authorize the sending mailbox for `https://www.googleapis.com/auth/gmail.send` using the authorization-code flow with `access_type=offline`. Exchange the code for a refresh token. If the account has already granted access and Google omits the refresh token, request consent again. Set `client_id`, `client_secret` where required by your OAuth client, `refresh_token`, and `sender` to the authorized mailbox or verified send-as address. A Google app in external testing can have short-lived refresh tokens. Use the consent configuration appropriate for sustained delivery. [Google offline authorization](https://developers.google.com/identity/protocols/oauth2/web-server#offline) · [Gmail sending guide](https://developers.google.com/workspace/gmail/api/guides/sending)

### Outlook.com and Microsoft 365

Register an Entra application that supports the sending account type. Grant the delegated Microsoft Graph permission `Mail.Send`. Have the sender authorize `offline_access https://graph.microsoft.com/Mail.Send` through the authorization-code flow and exchange the code for a refresh token. Set `client_id`, `refresh_token` and, for a confidential client, `client_secret`. Use the tenant GUID for a single-organization application. Use `consumers` for a personal Microsoft account or `common` for a compatible multi-account application. Organizational consent policies still apply. SentinelGrid sends as the authorized mailbox using Graph. [Microsoft authorization flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow) · [Microsoft Graph sendMail](https://learn.microsoft.com/en-us/graph/api/user-sendmail)

Mail providers can deliver to addresses at other domains. Each route targets one recipient. Add another route for independent retry and delivery status.

## Alert and retry behavior

- Every 15 seconds the server evaluates the newest sample for each enrolled host. Each host uses its customer defaults or its host override, including on fleet-wide routes. The initial triggers are one failed TCP check, CPU, memory or root disk usage at or above 90%, and a heartbeat older than two minutes. Additional configurable triggers cover slow TCP connections, port flapping and projected disk exhaustion. Edit the rules under **Alert thresholds** or **Host thresholds** on the dashboard. A fresh healthy observation sends a recovery. A host awaiting its first sample appears in the summary.
- Changes are grouped per route and evaluation. An unchanged incident generates one opening notice. Periodic summaries give the current host counts and list open conditions. Older active measurements stay marked as last known while a host is offline. A removed host or check is retired silently. The evaluation observes snapshots, so a failure that clears between evaluations can go unseen.
- Each route has a private disk queue and independently retries with exponential backoff. Rate-limit responses delay the next attempt. Failed messages survive restarts. Delayed summaries are replaced with the latest snapshot. The queue holds 512 messages per destination. If it fills, the server retains pending notices and reconciles the latest conditions as capacity returns. Intermediate transitions during saturation can be missed. Long notices show a count of additional conditions.
- Delivery is at least once for queued notifications while the route is active and the provider is accepting requests. A crash or network interruption after provider acceptance can cause a duplicate. Last accepted records the provider's acknowledgement. Inbox delivery and a Teams workflow's downstream chat action must be verified at the destination.
- Store backups of the server state and notification configuration privately. The state directory contains queues and rotated OAuth refresh credentials. Editing recipients, Teams URLs or tenant scope creates a fresh delivery state for that destination. Removing a route stops its worker. Restart after config changes. To disable all delivery, remove SG_NOTIFICATIONS from the service configuration and restart.
