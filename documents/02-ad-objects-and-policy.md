# AD Objects, Group Policy, Permissions, and Backup (CIS 158 / 258)

## Users, Groups, and Group Scope

- A **user account** is a security principal with a SID; a **group** is a collection of principals used to assign permissions once instead of per person.
- **Security groups** can receive permissions. **Distribution groups** cannot — they exist for email lists only. Exam questions hinge on that distinction.
- Group **scopes**:
  - **Global** — members from its own domain; can be used in any trusting domain; typically organized by role or job.
  - **Domain Local** — members can come from any domain; can only be assigned permissions in its own domain; typically tied to a resource.
  - **Universal** — members from any domain in the forest; usable anywhere in the forest; membership is stored in the Global Catalog, so keep universal group membership stable — every change replicates forest-wide.

## AGDLP — The Nesting Strategy

**AGDLP** is Microsoft's recommended group nesting pattern:

**A**ccounts go into **G**lobal groups, global groups go into **D**omain **L**ocal groups, and **P**ermissions are assigned to the domain local groups.

Why it works: the domain local group sits next to the resource ("DL — Print Lab, Modify"), the global group describes people ("G — Lab TAs"). People change jobs by moving between global groups; resources change by editing one domain local group. Permissions themselves almost never get re-assigned.

The fuller forest variant is **AGUDLP**, inserting Universal groups between Global and Domain Local in multi-domain forests. The exam answer for "how do you organize groups" is AGDLP; the day-one mistake it prevents is assigning permissions directly to user accounts.

## Group Policy Processing Order — LSDOU

Group Policy applies in a fixed order. **LSDOU**:

1. **L — Local** policy on the machine itself.
2. **S — Site** GPOs linked to the AD site.
3. **D — Domain** GPOs linked to the domain.
4. **OU — Organizational Unit** GPOs, from the outermost OU down to the OU holding the object.

Rules that decide exam questions:

- **Later wins.** Settings are applied in LSDOU order, and where settings conflict, the last one applied (usually the closest OU) wins.
- **Block Inheritance** on an OU stops GPOs from above applying — except **Enforced** (older name: No Override) GPOs, which cannot be blocked and apply last in their tier, so an enforced domain GPO beats a conflicting OU setting.
- Computer settings apply at boot/policy refresh to the computer object; user settings apply at logon to the user object. A GPO only affects the side (computer/user) whose object sits in its scope.
- **Loopback processing** makes user settings follow the *computer's* GPOs instead — the kiosk and lab-machine scenario.
- Verify before you theorize: `gpresult /r` (summary) and the Group Policy Results wizard show what actually applied.

Remember LSDOU as "the policy closest to the object speaks last" — Local, Site, Domain, OU.

## Password Policies and PSOs

- The **Default Domain Policy** holds the domain-wide password policy: minimum length, complexity, history, maximum/minimum age, lockout threshold/duration. There is effectively **one** such policy per domain — a password policy set in an OU GPO does *not* create a second domain password policy (a classic exam trap).
- **Fine-Grained Password Policies**, implemented as **Password Settings Objects (PSOs)**, are the exception: they target specific users or global groups (service accounts get longer, stricter settings; executives are not the use case, admins and services are). PSOs apply by precedence value — lowest wins when several target the same user.
- Account lockout is a denial-of-service trade: too low and attackers (or forgetful users) lock everyone out; too high and password guessing runs free. Pair lockout with alerting, not hope.

## Shares vs NTFS — Most Restrictive Wins

Two permission layers guard a shared folder:

- **Share permissions** (Full Control / Change / Read) apply only to access *over the network*.
- **NTFS permissions** (Full Control, Modify, Read & Execute, Write, Read) apply both locally and over the network, and are far more granular.

**Effective permission rule: when access comes over the network, the effective permission is the MOST RESTRICTIVE of the share permission and the NTFS permission.** Access on the console sees NTFS only.

**Worked mini-example:** the share grants the group **Change**; NTFS grants the same group **Read**. Over the network, effective = **Read** — most restrictive wins. Flip it (share = Read, NTFS = Modify) and effective is still **Read**. Only when *both* layers allow a right does the user actually have it over the network.

Within one layer, permissions combine differently: multiple NTFS *allows* accumulate (least restrictive of the allows), but an explicit **Deny** beats any Allow. Best practice that follows from AGDLP: share permission broad (e.g. Authenticated Users: Change, or Full Control), and do the real governing in NTFS assigned to domain local groups.

## Backup Types and the 3-2-1 Rule

- **Full** — everything selected. Slowest to take, simplest to restore.
- **Incremental** — everything changed since the **last backup of any kind**. Fast backups; restore needs the full plus every incremental in the chain, in order.
- **Differential** — everything changed since the **last full**. Restore needs only the full plus the latest differential; backups grow through the week.
- **Copy / mirror / snapshot variants** exist, but full/incremental/differential is the exam triangle — know which baseline each type measures from.

**3-2-1 rule:** keep **3** copies of the data, on **2** different media or systems, with **1** copy **offsite** (or offline/immutable — the ransomware-era reading). A backup that has never been test-restored is a hypothesis, not a backup. Server Backup, VSS snapshots, and replication are tools in this story, not synonyms for it (see the filesystems and data doc).

Hands-on cross-ref: navi-syskit carries the Windows Server cheat material for AD, DNS, DHCP, and GPO tasks — this doc is the why behind those steps.
