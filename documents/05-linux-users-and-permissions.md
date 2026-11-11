# Linux Users, Groups, and Permissions (CIS 141 / 142)

## The Account Files

- **/etc/passwd** — world-readable account list, seven colon-separated fields per line: `username : password-placeholder (x) : UID : GID : comment/GECOS : home directory : login shell`. The `x` means "the real hash lives in /etc/shadow." UID 0 is root — the number is the power, not the name. Regular users start at UID 1000 on Debian/RHEL-family systems; system accounts sit below.
- **/etc/shadow** — root-only. Nine fields: username, password hash (`!` = locked, `*` = no password login possible, `$6$...` = a SHA-512 crypt hash; modern systems may show `$y$` for yescrypt), then aging fields — last change, minimum days, maximum days, warning days, inactivity, account expiry, reserved. Shadow is why /etc/passwd can be public: names are not secrets, hashes are.
- **/etc/group** — group name, placeholder, GID, member list. A user's *primary* group comes from /etc/passwd; /etc/group lists *supplementary* memberships.
- Management commands: `useradd`, `usermod`, `userdel`, `groupadd`, `passwd` (set/change passwords), `id` (show who you are — UID, GID, groups), `groups`, `who`/`w` (who is logged in). Prefer these tools over hand-editing the files; they keep passwd/shadow/group consistent together.

## The sudo Model

**sudo** lets a permitted user run a specific command *as root* (or another user), authenticated with their **own** password, and **logged**. The policy file is **/etc/sudoers**, edited only with **`visudo`** — it syntax-checks before saving, because a broken sudoers locks everyone out of privilege.

- Membership in the **sudo** group (Debian/Ubuntu) or **wheel** group (RHEL family) is the usual grant.
- `sudo -i` gives a root login shell; plain `sudo <cmd>` runs one command. Principle of least privilege: sudo per command, not a permanent root shell.
- The security difference from sharing the root password: accountability. sudo logs *who* ran *what*; a shared password logs nothing attributable.

## Permission Bits — rwx and the Special Three

Every file has an owner, a group, and three permission triples — **user (owner), group, others** — each holding **r**ead (4), **w**rite (2), e**x**ecute (1). `ls -l` shows them as `-rwxr-xr-x`; a leading `d` marks a directory. On directories the bits mean: read = list names, write = create/delete entries, execute = traverse/enter.

**Numeric (octal) mode:** add each triple: 7 = rwx, 6 = rw-, 5 = r-x, 4 = r--, 0 = ---. So **755** = rwxr-xr-x and **644** = rw-r--r--. Commands: `chmod 755 file`, `chmod u+x script`, `chown user:group file`.

**Special bits** — a fourth octal digit in front (setuid 4, setgid 2, sticky 1):

- **setuid (4755, shown as `rws`)** — the program runs with the *file owner's* privileges, whoever launches it. Classic example: **/usr/bin/passwd** is setuid root, so an ordinary user can run it to update /etc/shadow — a file they cannot touch directly. Setuid root binaries are prime audit targets for exactly this reason.
- **setgid (2755, `rws` in the group triple)** — programs run with the file's group; on a **directory**, new files inherit the directory's group — the shared-team-folder mechanism.
- **sticky bit (1777, trailing `t`)** — on a directory, only a file's owner (or root) may delete it. Classic example: **/tmp** is `1777` — world-writable so everyone can work there, sticky so nobody can delete anyone else's files.

## umask Math — Worked Example

New files start from base modes — **666 for files, 777 for directories** (a plain file is never born executable) — and the **umask** *removes* permissions. The operation is bitwise (base AND NOT-umask), which behaves like subtraction when no borrow crosses a bit.

**umask 022:** file = 666 − 022 = **644** (rw-r--r--); directory = 777 − 022 = **755** (rwxr-xr-x). This is the common default: owner writes, everyone reads.

**umask 027:** file = 666 − 027 = **640** (rw-r-----); directory = 777 − 027 = **750** (rwxr-x---). Others get nothing — the privacy-conscious setting.

Exam habit: always compute *both* the file and the directory result when a **umask** question appears; "files and directories differ" is the point being tested. Check the current value with `umask`; set it per session with `umask 027` or persistently in shell startup files.

Hands-on cross-ref: navi-linuxkit does this permission and umask math live, with drills — this doc explains the model its answers come from.
