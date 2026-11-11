# Filesystems, Partitions, RAID, and Data Protection (CIS 113 / 141 / 158)

## Windows Filesystems, Honestly Compared

- **FAT32** — ancient, universally readable, and limited: **maximum file size 4 GB**, maximum volume 2 TB in theory (Windows will only format 32 GB). Right for small removable media and firmware partitions; wrong for anything modern and large.
- **exFAT** — Microsoft's removable-media successor: no 4 GB file cap, light structure, no journaling and no permissions. The default for large USB drives and SD cards shared between systems.
- **NTFS** — the Windows standard: journaling (survives crashes with metadata intact), ACL permissions, EFS encryption, compression, quotas, large files and volumes. Windows boots from NTFS.
- **ReFS** (Resilient File System) — integrity checksums and self-healing on mirrored/parity spaces, aimed at large data volumes and virtualization storage. Honest trade: historically it cannot be a boot volume and drops some NTFS features (EFS, compression) — a data-volume specialist, not a replacement.

## Linux Filesystems, at Concept Level

- **ext4** — the long-standing default: journaling, stable, understood by every recovery tool. The safe answer.
- **XFS** — built for very large files and high parallel throughput; the RHEL-family default. Shrinking an XFS filesystem is not supported — grow-only planning matters.
- **btrfs** — copy-on-write with built-in snapshots, checksums, and subvolumes; powerful data-integrity story, more moving parts to understand before trusting it in production.
- Shared concepts: a **journal** records intent before changes (fast, consistent crash recovery); mounting attaches a filesystem into the tree (below); Linux permissions and ownership (see the users-and-permissions doc) are enforced by the filesystem.

## RAID Levels — Failure Math in Words

RAID (redundant array of independent disks) trades disks for speed, safety, or both. The failure math is the exam:

- **RAID 0 — striping, minimum 2 disks.** Data split across all disks. Capacity = all of it; speed excellent; **redundancy zero — any one disk dying kills the whole array.** RAID 0 is not really "redundant" anything.
- **RAID 1 — mirroring, minimum 2 disks.** Every write goes to both. **Capacity = one disk's worth (50% with two disks); survives any one disk failure** (with two disks, only one can fail).
- **RAID 5 — striping with distributed parity, minimum 3 disks.** Parity blocks rotate across all disks. **Capacity = (n − 1) disks; survives exactly one disk failure.** RAID 5 with four 1 TB disks = 3 TB usable, one disk may die.
- **RAID 6 — dual distributed parity, minimum 4 disks.** **Capacity = (n − 2) disks; survives any two disk failures.** The premium is worth it on large, slow-rebuilding arrays, where a second failure during a RAID 5 rebuild is the realistic killer.
- **RAID 10 (1+0) — mirrored pairs, then striped, minimum 4 disks (always an even count).** **Capacity = 50%;** survives at least one failure, and can survive one per mirror pair — but not both members of any single pair. Fast rebuilds; the performance-and-safety favorite when disks are cheap enough.

RAID is **availability, not backup**: it does nothing against deletion, ransomware, corruption faithfully mirrored to both disks, theft, or fire.

## Partitions — MBR vs GPT

- **MBR (Master Boot Record)** — the legacy scheme: **maximum disk 2 TB, maximum 4 primary partitions** (workaround: one extended partition holding logical drives). One small boot record is a single point of failure.
- **GPT (GUID Partition Table)** — the modern scheme: effectively no practical partition count limit (128 by Windows default), disks far beyond 2 TB, redundant partition tables (header + backup) with CRC checks. Required to boot in UEFI mode on Windows.
- On Linux a filesystem is **mounted** into the tree at a directory (mount point); **/etc/fstab** lists mounts to perform at boot — fields: device, mount point, filesystem type, options, dump, fsck order. A typo in fstab can stop a boot: edit, then `mount -a` to test before rebooting.

## Backup vs Snapshot vs Replication

Three words that are not synonyms:

- **Backup** — an independent copy, kept separately, restorable to a point in time. The only one of the three that saves you from deletion, corruption, and ransomware that happened yesterday — *if* an older copy is kept. Governed by retention and the 3-2-1 rule (see the AD objects and policy doc).
- **Snapshot** — a point-in-time *state* recorded inside the same system (VM snapshots, VSS, btrfs snapshots). Instant and space-efficient, brilliant for "just before this risky change," worthless if the underlying storage dies — the snapshot lives there too. Old snapshots also quietly eat disk.
- **Replication** — keeping a *live copy in sync* on another system (AD replication, DFS Replication, storage replication). Buys availability and site resilience; propagates deletions and corruption faithfully and quickly. Replication is why the second copy is current — including its mistakes.

Exam synthesis: snapshot for rollback, replication for uptime, backup for disaster — a complete design usually uses all three and confuses none of them.

Hands-on cross-ref: navi-syskit carries the Windows storage and backup planning material; navi-linuxkit covers the mount and permissions side at the keyboard.
