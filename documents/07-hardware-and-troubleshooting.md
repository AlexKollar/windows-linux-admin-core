# Hardware and Troubleshooting (CIS 113 — A+ Core)

## CPU, RAM, and Storage Families

- **CPU** — socket type must match the motherboard (a CPU physically fits one socket family); core/thread counts, cache, and integrated graphics matter more to selection than raw GHz. Cooling is part of the spec: a CPU under an undersized cooler throttles and *looks* like a software problem.
- **RAM generations — the numbers that matter:**
  - **DDR3** — 240 pins, **1.5 V** (1.35 V for DDR3L).
  - **DDR4** — 288 pins, **1.2 V**.
  - **DDR5** — 288 pins, **1.1 V**, on-die ECC, power management moved onto the module.
  - Generations are **not interchangeable** — the notch sits in a different place so the wrong stick physically will not seat. Speeds are rated in MT/s; matched pairs enable dual-channel bandwidth.
- **Storage interfaces** — SATA III is 6 Gb/s (~550 MB/s real ceiling) for spinning disks and 2.5" SSDs. **NVMe** drives speak PCIe directly (multiple GB/s, generation-dependent) in M.2 or U.2 form factors; an M.2 slot may carry SATA or NVMe, so check which the slot speaks. HDDs still win on cost per terabyte for bulk storage.

## PSU Sizing Thinking

Size the power supply to the *real* simultaneous load, not the sum of marketing numbers: add CPU and GPU rated draw, allow for drives/fans/USB, then buy headroom — a quality unit loafing at 50–70% of rating runs cooler, quieter, and more efficiently than a bargain unit at 95%. **80 PLUS** tiers (Bronze → Titanium) rate efficiency, not quality by themselves. A system that dies under load but idles fine is pointing at the PSU or thermals before anything exotic.

## Boot Order, POST, and Beep Honesty

At power-on the firmware runs **POST** (power-on self-test): CPU, memory, video, and boot devices are checked, then the firmware walks the **boot order** to find a bootloader. A machine that cannot find one says so ("no boot device") — that message is firmware reporting an empty list or a dead drive, not an OS error.

**Beep codes — the honesty rule:** there is **no universal beep standard**. Patterns differ by firmware vendor (AMI, Award, Phoenix, OEM variants). One long + two/three short often suggests video on several vendors, and repeated memory beeps exist on many boards — but the only authoritative decoder is that board's manual, and modern boards increasingly use debug LEDs or two-digit POST code displays instead. In the lab: reseat RAM first for no-POST-no-display (the single most common cause), then consult the manual for the exact pattern.

## BSOD-Name Decoder

A Windows stop code names a suspect, not a verdict — each implicates a first place to look:

- **IRQL_NOT_LESS_OR_EQUAL** — a driver touched memory at too high an interrupt level. First suspect: a recently changed **driver** (or failing RAM underneath it).
- **MEMORY_MANAGEMENT** — the memory manager found corruption. First suspect: **RAM** — run a memory test before reinstalling anything.
- **INACCESSIBLE_BOOT_DEVICE** — Windows lost its boot volume mid-start. Storage driver change, controller mode flipped in firmware (AHCI/RAID), or a failing drive.
- **CRITICAL_PROCESS_DIED** — a required system process was terminated, often by file corruption or a bad update. Check disk and system-file integrity.
- **KERNEL_SECURITY_CHECK_FAILURE** — a kernel data structure failed validation: driver corruption, memory errors, or disk errors — start with drivers and memory.
- **PAGE_FAULT_IN_NONPAGED_AREA** — invalid memory reference in code that must stay resident: drivers and RAM again, the usual pair.
- **SYSTEM_SERVICE_EXCEPTION** — an exception in kernel code while handling a service call; a named driver file on the screen, when present, is the lead.
- **DRIVER_POWER_STATE_FAILURE** — a device did not handle a power transition: sleep/wake problems point at device drivers.

Method: note the code and any named file, ask what changed, test memory and disk, then drivers — in that order of cheapness.

## The CompTIA Troubleshooting Methodology — Six Steps, In Order

1. **Identify the problem** — gather information, question users, identify symptoms, determine what changed, check logs.
2. **Establish a theory of probable cause** — question the obvious, consider multiple approaches, start with the likeliest explanation.
3. **Test the theory** to determine the cause — if the theory fails, form a new one or escalate.
4. **Establish a plan of action** to resolve the problem and identify potential effects.
5. **Implement the solution or escalate** as necessary.
6. **Verify full system functionality** and, if applicable, implement **preventive measures**.

**Document findings, actions, and outcomes throughout** — CompTIA treats documentation as wrapped around all six steps, and real shops treat it as the difference between a fix and folklore. Exam traps live in the ordering: you never implement before testing the theory, and "verify" is never the step you skip when the symptom disappears early.

Hands-on cross-ref: navi-syskit quizzes these BSOD codes, hardware numbers, and methodology steps in drill form — this doc is where the answers are explained.
