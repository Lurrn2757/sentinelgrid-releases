# SentinelGrid host agent · standalone 0.2.1

Use the archive for the host CPU (x86_64 = amd64, aarch64 = arm64). Get a dedicated host key from the monitoring administrator. Extract the archive and run this command from the extracted directory.

```sh
sudo ./install.sh https://monitor.example.com 22,443 < web-01.key
```

Replace the URL with your own SentinelGrid HTTPS address. Replace the port list with the TCP services expected on this host or omit it to collect inventory only. This installs an unprivileged systemd service, enables it at boot, and starts reporting. Check `systemctl status sentinelgrid-agent` or `journalctl -u sentinelgrid-agent`. Use the same command to change ports, server, or host key. For an upgrade with the existing configuration run `sudo ./install.sh`.

The agent sends CPU, memory, root disk usage, uptime, load, kernel release, network byte totals, and TCP results every 30 seconds. CPU is measured between observations. Network totals exclude loopback. Listening inventory reports up to 128 distinct ports visible in the agent's Linux network namespace, across IPv4 and IPv6. Active checks attempt a connection on the local loopback addresses for up to 32 configured ports. A listener bound exclusively to another address can appear in inventory and fail the loopback check. Success measures TCP connection establishment. HTTP status, certificate age and application readiness require separate application checks.

The local spool retains up to 240 samples during an outage and replays in batches after reconnection. On overflow it drops the oldest sample. The server keeps the newest 120 per host. A server or tenant identity change uses a separate spool. Keep the system CA store current. A private CA must be trusted by the host.

For a foreground run, protect the key file with chmod 600 and use the following command. Add -once for one sampling and delivery cycle.

```sh
./sentinelgrid-agent -server https://monitor.example.com -key-file ./web-01.key -ports 22,443
```
