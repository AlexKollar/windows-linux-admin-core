# Windows Client Administration (CIS 113 / 155)

## The Admin Toolset — Which Tool Answers Which Question

- **Event Viewer** — the Windows logbook. Channels that matter: **Application** (program errors), **Security** (logon success/failure and auditing — event 4624 success, 4625 failure), **System** (drivers, services, shutdowns), and **Setup**. Custom views and filtering by level/source are how you read it without drowning. When a user says "it crashed around 2," Event Viewer is the first honest witness.
- **Task Manager** — the quick triage view: per-process CPU/memory/disk/network, startup impact, user sessions. Honest framing: it shows *that* something hurts, not *why*.
- **Resource Monitor (resmon)** and **Performance Monitor (perfmon)** — the "why" tools: which process holds a file, which disk queue is saturated, counters over time. Reach for these when Task Manager's summary is not enough.
- **Services (services.msc)** — start/stop, startup type (Automatic, Automatic Delayed Start, Manual, Disabled), and the log-on account a service runs as. A service set to Disabled cannot be started by anything — a common root cause hiding in plain sight.
- **Disk Management (diskmgmt.msc)** — initialize disks (MBR/GPT), create/format volumes, assign drive letters, extend/shrink partitions, read disk health states like Offline or Foreign.
- **Computer Management** — the umbrella console: local users and groups, shared folders, Disk Management, Event Viewer, Device Manager in one place.
- **Device Manager** — driver state at a glance: the yellow exclamation point is a device with a problem worth opening first.
- **Registry Editor (regedit) — caution label.** The registry is the configuration database (hives such as HKLM and HKCU). Honest admin practice: export before you edit, change one thing, document it, and prefer Group Policy or Settings over hand-editing — a wrong registry edit can make a machine unbootable, and there is no undo button.

Supporting cast: `msconfig` (boot and startup control, Safe Boot toggle), `msinfo32` (system inventory in one page), Control Panel vs Settings (both exist; Settings is where new controls land).

## Boot Process and Recovery Options

Modern Windows boots: firmware (UEFI) → Windows Boot Manager → `winload` → kernel and boot-start drivers → Session Manager and Winlogon → logon screen. Where it stops tells you the layer that's broken; "no boot device" is firmware talking, a spinning circle after the logo is Windows talking.

**Windows Recovery Environment (WinRE)** options, in escalating order:

- **Startup Repair** — automated fixes for boot files and configuration.
- **System Restore** — rolls system files, drivers, and registry back to a restore point; does not touch user files, and does not remove malware hiding in them.
- **System Image Recovery** — restores a full captured image.
- **Safe Mode** — minimal drivers and services: if it boots in Safe Mode, a third-party driver or service is the suspect. Safe Mode with Networking adds the NIC driver for downloads.
- **Uninstall Updates** — back out a quality or feature update that broke boot.
- **Command Prompt** — the manual shelf: `bootrec`, `bcdedit`, `sfc /scannow`, `DISM` repairs.
- **Reset This PC** — keep files or remove everything; the last resort before reimaging.

## Windows Security Features

- **Microsoft Defender Antivirus** — real-time protection, cloud-delivered protection, tamper protection (so malware cannot simply switch it off). Controlled Folder Access is its ransomware tripwire for user folders.
- **BitLocker** — full-volume encryption. Keys are protected by the **TPM** and can escrow to Active Directory/Azure AD. Lose both the TPM path and the recovery key and the data is gone by design — that is the feature working, not failing.
- **Windows Firewall with Advanced Security** — three profiles: **Domain** (on the corporate network), **Private** (trusted home/work), **Public** (coffee shop — most restrictive by default). Rules are inbound/outbound, per program, port, or profile.
- **User Account Control (UAC)** — standard users run unprivileged; elevation is an explicit prompt. Admin Approval Mode means even administrators approve their own elevation.
- **Windows Hello / SmartScreen / Secure Boot** — biometric or PIN sign-in tied to the device, reputation checks on downloads and sites, and firmware-level verification that the boot chain is signed.

## Update Rings, at Concept Level

Windows Update is managed as **rings** — groups of devices that receive updates on different delays. A cautious layout: a small pilot ring gets feature and quality updates first, a broad production ring follows days or weeks later after the pilot proves them, and critical servers sit in the slowest ring with scheduled maintenance windows. Quality updates (monthly, cumulative, security) and feature updates (larger, version-changing) ride the same rings at different cadences. The concept to carry: rings turn "Patch Tuesday broke everything" into "Patch Tuesday broke the pilot ring, and we pause the rollout" — deferral is a control, not neglect.

Hands-on cross-ref: navi-syskit drills the A+ side of this material — boot failures, BSOD codes, and tool choices in quiz form.
