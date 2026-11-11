# Windows Server and Active Directory (CIS 158 / 258)

## Windows Server Roles at a Glance

A **role** is a job you give a server; a **feature** is a supporting capability. Roles the A.A.S. labs return to constantly:

- **AD DS (Active Directory Domain Services)** — the identity store and authentication authority for the domain.
- **DNS Server** — name resolution. Active Directory cannot function without DNS: domain controllers advertise services with SRV records, and clients find a DC by querying DNS first.
- **DHCP Server** — hands out addresses and options (gateway, DNS servers, domain name) automatically.
- **File and Storage Services** — shares, quotas, deduplication, storage pools.
- **Print Services, Web Server (IIS), Remote Desktop Services, WSUS (updates)** — common supporting roles.
- **Group Policy Management** ships with AD DS and is how configuration is pushed centrally.

Exam framing: installing a role is not configuring it. AD DS installation only finishes when the server is **promoted to a domain controller (DC)** — that promotion creates (or joins) a domain.

## AD DS Anatomy — Forest, Tree, Domain

- **Object** — the unit of the directory: a user, computer, group, printer, or OU, each with attributes.
- **Organizational Unit (OU)** — a container *inside* a domain used to organize objects, delegate administration, and apply Group Policy. An OU is not a security boundary.
- **Domain** — a partition of the directory with one replicated database, one set of domain controllers, and shared policies. The domain *is* an administrative and replication boundary. Example: `corp.example.local`.
- **Tree** — a domain plus its child domains in one continuous DNS namespace (`corp.example.local`, `lab.corp.example.local`). Domains in a tree trust each other automatically (two-way, transitive).
- **Forest** — one or more trees sharing one schema, one configuration partition, and one global catalog. The forest is the true **security boundary** of Active Directory. The first domain created is the **forest root domain**.
- **Schema** — the definitions of every object class and attribute the forest allows. Schema changes are rare, forest-wide, and effectively permanent — treat them accordingly.
- **Global Catalog (GC)** — a partial, forest-wide index held by designated DCs; it answers "where in the forest is this object?" and is required for logon in multi-domain forests.
- **Site** — a physical/topological object (next section), not part of the DNS namespace.

Trusts to name: parent–child and tree-root trusts are automatic, two-way, and transitive. **External** and **forest** trusts are created manually and are not automatically transitive.

## Sites and Replication, at Concept Level

A **site** maps to a well-connected physical location (usually a LAN/subnet). Sites control two things: **replication** (DCs replicate eagerly inside a site; between sites, replication is scheduled and compressed over site links) and **logon locality** (a client should authenticate to a DC in its own site). Hands-on rule: a new subnet that is not mapped to a site is why logons mysteriously cross a WAN link.

## The Five FSMO Roles — Named and Placed

Flexible Single Master Operations (FSMO, "fizz-mo") roles exist where only one DC may do a job at a time.

**Forest-wide — one per forest (two roles):**

1. **Schema Master** — the only DC that may change the schema.
2. **Domain Naming Master** — the only DC that may add or remove domains in the forest.

**Domain-wide — one per domain (three roles):**

3. **RID Master** — hands out pools of Relative IDs to DCs. Every security principal's SID is the domain SID plus a RID, so RIDs must never be issued twice.
4. **PDC Emulator** — acts as the domain's authority for password-change urgency (failed logons are re-checked here first), Group Policy editing by default, and the domain's primary time source.
5. **Infrastructure Master** — keeps cross-domain group-to-user references current when objects move between domains in the same forest.

Memory hook: the forest owns the **names and the shapes** (Domain Naming, Schema); each domain owns **IDs, passwords/time, and references** (RID, PDC Emulator, Infrastructure). Any DC can *hold* a role; roles can be **transferred** (graceful, both online) or **seized** (forced, old holder must never return).

## DNS and DHCP Server Roles

**DNS for AD:** zones are often **AD-integrated** — stored in the directory and replicated with it, allowing secure dynamic updates. Key records: A/AAAA (hosts), CNAME (aliases), MX (mail), and the SRV records (`_ldap._tcp...`) clients use to locate DCs. If "the domain is broken," check DNS before anything else: most AD failures are DNS failures wearing a costume.

**DHCP sequence — DORA:** the client **D**iscovers (broadcast), the server **O**ffers an address, the client **R**equests it, the server **A**cknowledges. A **scope** is the pool for one subnet; **reservations** pin an address to a MAC; **lease duration** balances stability against churn. A DHCP server in a domain must be **authorized** in AD before it will serve — a rogue-DHCP defense.

## The Join / OU / GPO Workflow, in Words

1. Build the server or workstation with a static or reserved address and point its DNS at a domain DNS server — never at the ISP's resolver.
2. **Join the domain** (Settings → Access work or school, or System Properties → Computer Name). The machine gets a computer account and restarts.
3. **Move the computer object** out of the default Computers container into the OU that matches its role and location. Containers are not OUs — you cannot link a GPO to the default Computers container, which is exactly why this step matters.
4. The OU receives **Group Policy Objects (GPOs)** linked from above (site, domain, OU — see the admin-core doc on policy processing order, LSDOU).
5. The admin manages users and groups in **Active Directory Users and Computers (ADUC)**, policy in the **Group Policy Management Console (GPMC)**, and verifies results with `gpresult` / Group Policy Results before blaming the network.

Hands-on cross-ref: navi-syskit carries the Windows admin reference for these same tasks in command form — use it at the bench, use this pack to understand what the bench work means.
