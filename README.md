# SentinelGrid downloads

Install the server, install an agent, connect it and monitor your infrastructure.
This repository holds customer documentation and packaged releases. Application
source and CI remain in a separate private development repository. GitHub's
source archives for this repository contain only these public documents.

Features include host and Proxmox metrics, PostgreSQL ingestion, Grafana,
Zabbix alert lifecycles, Telegram controls, optional AI reports and email alerts,
durable offline metrics, and verified backup/recovery. Integrations require
your own credentials. Server installation does not install Zabbix itself; it
preserves the existing optional Zabbix polling/webhook integration.

## Install

Supported: Ubuntu 22.04/24.04 and Debian 12, amd64/arm64, with systemd. The
server requires Docker Engine and Compose >=2.24. Download packages from
[SentinelGrid releases](https://github.com/Lurrn2757/sentinelgrid-releases/releases).
The source repository stays private; installation requires no source checkout,
Python package downloads or registry authentication.

On a supported server, install prerequisites from the signed OS repositories,
then download and verify the pinned installer before running it:

~~~bash
sudo apt-get update -qq
sudo apt-get install -y curl ca-certificates openssl python3
mkdir -m 700 sentinelgrid-install && cd sentinelgrid-install
base=https://github.com/Lurrn2757/sentinelgrid-releases/releases/download/v0.1.3
curl --proto '=https' --proto-redir '=https' -fsSL "$base/checksums.txt" -o checksums.txt
curl --proto '=https' --proto-redir '=https' -fsSL "$base/install.sh" -o install.sh
awk '$2 == "install.sh" {print}' checksums.txt | sha256sum --check --status &&
  sudo bash install.sh --version 0.1.3 --install-docker
~~~

The installer downloads and verifies the server bundle and the image archive for
this host, loads the images, generates private credentials, starts the stack and
checks readiness. Docker installation requires confirmation. Grafana credentials
are in `/etc/sentinelgrid/sentinelgrid.env` (0600). Ports bind to loopback by
default. Set trusted interface addresses in the config and restart, or explicitly
pass `--public-bind` during install for a protected LAN/VPN. A remote connection
needs an existing TLS reverse proxy or a trusted VPN/LAN.

Create a private connection kit on the server:

~~~bash
sudo sentinelgrid agent add --name web-01 --arch amd64 \
  --server https://monitor.example.com --output /root/web-01-kit
~~~

Use `--arch arm64` for ARM hosts. The server URL must reach the ingestion API,
usually port 8000 behind your HTTPS proxy. For a trusted LAN/VPN, use an HTTP
URL with `--allow-http`. Transfer the entire kit securely to the target host,
keep its directory private (0700) and files private (0600), and run:

~~~bash
cd /path/to/web-01-kit
sudo bash connect.sh
~~~

The installer registers the host, sends its first metric, starts the service and
checks that it is active. The token permits one registration and expires after
one hour. Delete both copies of the kit after use. Each agent receives its own
credential; no server admin key is shipped. Prerequisites are `python3`, `curl`,
`openssl`, CA certificates, `tar`, `useradd`, `runuser` and systemd. Python is used
for installation validation; the monitoring agent carries its own runtime.

For manual enrollment, the verified `install-agent.sh --server URL` prompts for
a token without echoing it. Use `--token-file PATH` for a private unattended
input, or `--no-enroll` to prepare without registration/startup. Obtain tokens
with `sudo sentinelgrid token create --name HOST --expires-in 1 --max-uses 1`.
The older offline wheel format remains available for development fixtures.

Check connections with `sudo -u sentinelgrid-agent sentinelgrid-agent status`
and `sudo systemctl status sentinelgrid-agent`. If initial delivery fails after
registration, the service retries with durable buffering and the installer exits
nonzero. If registration fails, correct the URL/network or obtain a fresh token,
register as the service user with `--token-stdin`, then start the service.

Checksums from the same HTTPS origin detect corruption relative to the manifest;
they are not signatures. Kits also pin the agent checksum from the installed
server release. Machine administrators can inspect bytecode and binaries; this
packaging removes loose application source, not all possibility of inspection.
See the release assets for installers, offline image archives and agent packages.

`sentinelgrid update` and rollback remain planned. Installers refuse existing
installations and persistent data. Back up configuration and data before manual
migration or uninstall. Uninstall preserves `/var/lib/sentinelgrid` unless
`--purge` is explicitly passed. All existing monitoring features remain present;
Telegram, Zabbix, Proxmox, Gemini and email need their integration credentials.

