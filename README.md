# SentinelGrid 0.2.1

Self-hosted Linux monitoring with customer dashboards, configurable alerts and recurring Microsoft Teams updates.

[Download 0.2.1](https://github.com/Lurrn2757/sentinelgrid-releases/releases/tag/v0.2.1) · [Release notes](releases/0.2.1.md) · [Server setup](docs/0.2.1/server.md) · [Agent setup](docs/0.2.1/agent.md) · [Notifications](docs/0.2.1/NOTIFICATIONS.md)

## One server, one agent

The server hosts the dashboard, receives telemetry and delivers notifications. Each monitored machine runs an outbound agent. Customers see their assigned hosts and can tune alert thresholds. Administrators see the fleet.

Linux AMD64 and ARM64 downloads include the executable, systemd installer, service configuration and setup guide. The server package is about 4 MB compressed and the agent about 3 MB.

## Included in 0.2.1

- CPU, memory, root disk, network totals, load, uptime, kernel and listening-port inventory.
- Active TCP connection checks, missing-heartbeat detection and offline telemetry replay.
- Customer defaults, host overrides and a preview of threshold changes against current readings.
- Slow TCP and flapping detection, disk-exhaustion forecasts and a recent host-change timeline.
- Gmail, Outlook, Telegram and Microsoft Teams group-chat alerts, recovery messages and scheduled fleet updates.
- Separate host credentials, scoped customer access and persistent notification retries.

## Download and install

Download the server archive for the monitoring server and the agent archive for each monitored host. Verify the selected files using [SHA256SUMS](downloads/0.2.1/SHA256SUMS), extract each archive and follow its README.md. Run `sudo ./install.sh` from the extracted server directory, configure HTTPS, then issue customer logins and individual host keys.

On a monitored host, run the installer with your address and the private key issued to that host.

```sh
sudo ./install.sh https://monitor.example.com 22,443 < web-01.key
```

Direct download copies are also available in [downloads/0.2.1](downloads/0.2.1).

## Upgrade and compatibility

Standalone preview users can back up their state and install the server update. Existing standalone agents remain compatible. Earlier Docker/PostgreSQL deployments use separate state and credentials. Follow the [migration notes](releases/0.2.1.md) when moving those hosts to this edition.

The standalone series has been exercised on Ubuntu kernel 6.8 and Linux 7.3, with ARM64 testing under emulation. Kernel 5.0.1 and native ARM64 production environments remain unverified. The server retains up to 120 samples per host. Provider accounts require credentials and a live delivery check after configuration.

This public repository contains packaged downloads and user documentation. Development source is maintained in private storage.
