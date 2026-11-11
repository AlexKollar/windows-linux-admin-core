# Linux Networking and Security (CIS 141 / 142)

## Interface and Socket Tools

- **`ip`** is the modern toolkit: `ip addr` (interfaces and addresses), `ip link` (up/down state), `ip route` (the routing table — "default via" is the gateway), `ip neigh` (the ARP/neighbor cache). The old `ifconfig`/`route` commands are deprecated; exams now expect `ip`.
- **`ss`** shows sockets: `ss -tlnp` = TCP, listening, numeric, with the owning process. It answers the core security question "what is this machine listening on, and what program owns it?" `netstat` is its retired predecessor.
- Supporting cast: `ping` (reachability + latency, ICMP), `dig` / `host` (DNS answers in detail), `traceroute` / `tracepath` (the path), `curl` (speak to a service and see exactly what it says back).

## Configuring the Network — netplan vs NetworkManager, at Concept Level

Two management styles dominate:

- **NetworkManager** — a daemon managing connections dynamically; the default on most desktops and many servers. Configure with `nmcli` / `nmtui` or the GUI; it suits machines whose networks change (laptops, Wi-Fi).
- **netplan** (Ubuntu/server world) — declarative YAML in `/etc/netplan/` describing the desired state; `netplan apply` renders it to NetworkManager or systemd-networkd underneath. Suits servers: configuration as a reviewable file.
- RHEL-family servers use NetworkManager with keyfiles (and historically ifcfg files). The concept that transfers: there is always one manager of record — hand-editing interface state underneath it (`ip addr add` directly) works until the manager reasserts its configuration.

## SSH — Client and Server Hardening, in Words

SSH configuration lives in `/etc/ssh/sshd_config` (server) and `~/.ssh/config` (per-user client conveniences). The hardening checklist, each item with its reason:

1. **Key-only authentication** — `PasswordAuthentication no` after keys are installed and tested. Kills password guessing entirely.
2. **`PermitRootLogin no`** (or `prohibit-password`) — root is reached by logging in as yourself and using sudo, so the log names a person.
3. **Limit who may connect** — `AllowUsers` / firewall source restrictions; a lab machine does not need the whole internet at its SSH door.
4. **Change the port?** Optional obscurity: it cuts background noise in the logs, it is *not* a security control by itself — say so honestly on an exam and in the lab.
5. **Disable what you don't use** — X11 forwarding, agent forwarding, empty passwords (never permitted).
6. **Fail2ban** or equivalent — watch the auth log and temporarily ban sources with repeated failures. A seatbelt on top of key-only auth, not a substitute.
7. After any edit: `sshd -t` to test the config *before* restarting, and keep one session open while you test the next — the classic way admins lock themselves out is skipping both.

## Firewalls — ufw and firewalld Concepts

- **ufw** (Uncomplicated Firewall, Debian/Ubuntu/Pop!_OS) — a friendly front end to the kernel packet filter: `ufw default deny incoming`, `ufw allow ssh`, `ufw enable`, `ufw status`. Model: default-deny inbound, allow named services, outbound open by default.
- **firewalld** (RHEL family) — **zones** (public, home, internal...) assigned to interfaces, with services/ports allowed per zone; runtime vs permanent configuration is its classic gotcha (a runtime rule vanishes at reload unless made permanent).
- Shared concepts beneath both: stateful filtering (replies to your outbound connections are allowed), rules are evaluated against new connections, and the firewall you forgot is off protects nothing — check `status` before trusting it.

## The Log Locations Map — Which File Answers Which Question

- **Authentication and privilege:** `/var/log/auth.log` (Debian family) / `/var/log/secure` (RHEL family) — logins, sudo, SSH. "Who got in, and how?" starts here.
- **Everything general:** the **journal** via `journalctl` (see the foundations doc), plus `/var/log/syslog` (Debian) / `/var/log/messages` (RHEL).
- **Boot:** `journalctl -b`, `dmesg` for kernel messages.
- **Packages:** `/var/log/apt/` or `/var/log/dnf.log` — "what changed on this box, and when?"
- **Services keep their own:** web servers under `/var/log/apache2/` or `/var/log/nginx/`, mail in `mail.log`.
- Cross-ref: navi-scribe is the chip that parses and flags exactly these auth and system logs — this map tells you which file to feed it.

## Scheduling — cron and systemd Timers

- **cron** runs commands on a calendar pattern: each user has a crontab (`crontab -e`), with five fields — minute, hour, day of month, month, day of week — then the command. `0 3 * * *` reads "03:00 every day." System jobs also live in `/etc/crontab` and `/etc/cron.d/`.
- **systemd timers** are the modern alternative: a `.timer` unit triggers a `.service`, gains journal logging, `Persistent=true` (run missed jobs after downtime — cron's gap that anacron patched), and inspection with `systemctl list-timers`.
- Exam framing: cron is universal and simple; timers integrate with systemd's logging and dependency model. Know the five cron fields cold — misreading field order is the most common cron failure.

Hands-on cross-ref: navi-linuxkit carries the hardening checklists in working form; navi-syskit covers the matching Windows-side services and firewall ideas.
