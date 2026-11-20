# SentinelGrid server · standalone 0.2.1

On an Ubuntu system with systemd, extract the server archive and run `sudo ./install.sh`. The installer starts the dashboard and ingestion service on 127.0.0.1:8080. Both use the same address. Forward an existing HTTPS reverse proxy to this port, forwarding Authorization and preserving the request path. For direct HTTPS, edit /etc/sentinelgrid-server/config with LISTEN=0.0.0.0:8443 and CERT and KEY paths. The service user needs read access to the certificate chain and private key. Restart with `sudo systemctl restart sentinelgrid-server` after changes or certificate renewal.

Create logins and a host credential on the server. Each login command prints a generated password. Issuing a credential for an existing name rotates it immediately.

```sh
sg() { sudo -u sentinelgrid-server /usr/local/bin/sentinelgrid-server -state /var/lib/sentinelgrid-server "$@"; }
sg login admin
sg login acme
umask 077
sg agent acme web-01 > web-01.key
```

Open your HTTPS address. Enter admin and its generated password at the browser prompt to see all servers. Give the customer the acme login and its password. Multiple hosts can belong to one customer. Deliver web-01.key privately to the operator installing the agent on that machine. Use separate keys for each host. The customer login can view that customer's hosts and edit that customer's alert settings. Provisioning and rotation require local operating-system access to the server state.

```sh
sg revoke-agent acme web-01
sg revoke-login acme
```

Revoking a host hides it from the fleet and disables uploads. Its retained history remains on disk. Reissuing the same tenant and host name resumes that identity. Revoking a customer login disables dashboard access while enrolled agents continue collecting. For customer offboarding, revoke their hosts as well.

The host agent requires outbound HTTPS only. If the server also needs monitoring, enroll it as a host. Check service logs with `journalctl -u sentinelgrid-server`. For a consistent private backup, stop this service, copy /var/lib/sentinelgrid-server including ownership and permissions, then restart. Run one server process per state directory on a local Linux filesystem.

The standalone server keeps the newest 120 samples per host, about one hour at the normal cadence. CPU trend displays the last 30. The initial heartbeat timeout is two minutes. CPU, memory and root disk thresholds each start at 90%. Port failures appear on the dashboard. Keep host clocks synchronized.

This first standalone edition uses its own data directory, agent protocol and dashboard. Existing 0.2 Python agents, historical PostgreSQL data, and Proxmox polling belong to the recovered deployment. The two editions can run separately.

For customer alerts, recovery notices and recurring Teams group-chat updates, follow [notification setup](NOTIFICATIONS.md).

## Tune alerts in the dashboard

Select **Alert thresholds** for customer defaults, or **Host thresholds** on a host card. The admin login can edit settings for any customer. A host override is a full snapshot. Select **Use customer defaults** to resume inheritance. New hosts inherit the customer's defaults automatically.

**Preview impact** compares proposed settings with the current readings for the affected hosts. **Save** persists the settings and applies them to the dashboard and every notification route, including the administrator's fleet-wide routes. A save can open or clear an incident at the next evaluation. Disabling a rule retires its open condition silently. Previously queued messages can still arrive. The form rejects stale edits from a second browser tab. Changes require no agent rollout or server restart.

- **Resource pressure and missing heartbeats.** Set separate CPU, memory and root disk percentages and the heartbeat timeout in seconds. Zero disables the associated rule. A timeout shorter than the normal 30-second sampling interval can flag a healthy host between reports.
- **Slow or intermittent TCP.** Set the consecutive failure count and a connection-latency limit. For example, three consecutive failures suppresses one-off failures. Four state changes in ten minutes flags a port repeatedly switching between open and closed, even if it is currently open. A recovered connection can remain flagged for flapping until earlier changes age out of the lookback. Checks use the host's loopback interface.
- **Disk exhaustion forecast.** Set a warning horizon, a lookback period and minimum trend fit. For example, a six-hour horizon, twenty-minute lookback and 80% minimum fit warns when observed growth projects a full root filesystem within six hours. The server fits a straight line to timestamped disk usage and checks the fraction of variation explained by the line (R²). It requires at least six observations spanning half the chosen window. Flat or inconsistent growth yields no prediction. Batch writes, cleanup and changes in workload can invalidate a projection.
- **Host change timeline.** Expand **Recent host changes** to see the latest twelve changes captured across retained samples, including newly observed or missing listener ports, a changed kernel release or a decrease in uptime suggesting a reboot. Times are collection times in UTC. Listening inventory is sampled and capped at 128 ports.
  
Latency, flapping and forecast notifications start disabled. A value of zero disables any alert trigger. The lookback windows and fit setting take positive values. Time-series rules discard a streak or fit across gaps longer than two minutes. Backup of the server state directory includes customer and host rules.
