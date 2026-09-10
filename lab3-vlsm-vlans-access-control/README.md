**HACKING THE WORKFORCE**

**Lab 3 --- The Subnetting Puzzle & Access Control**

*Week 3 · OSI Layer 3 (VLSM) · Worksheet & Submission*

*Self-directed edition --- built from the program\'s Week 3 preview,
since full instructions weren\'t available outside the cohort platform.*

  **Name**      **Date**    **Environment used**   **Time spent**
  ------------- ----------- ---------------------- ----------------
  Omar Swoope   8/10/2026   Packet tracer          2.5

Scenario
========

*A small business is building out its office network from a single
block: 192.168.10.0/24. It needs four separate subnets ---
Production/Staff, VOIP phones, Guest Wi-Fi, and Management
(switches/APs) --- each sized differently, with strict access control
between them. Your job: size each subnet with VLSM, carve up the /24
without overlap, decide which subnets may talk to which, and write the
policy that governs future subnet requests.*

Part A --- Sizing the Subnets (VLSM, pencil first)
==================================================

The host-bits method
--------------------

*Every subnet needs enough host bits to cover its device count, plus 2
reserved addresses (network + broadcast). The formula: 2\^h − 2 ≥ hosts
needed, where h = host bits. Find the SMALLEST h that satisfies each
department\'s requirement --- using more bits than necessary wastes
addresses.*

  **Host Bits (h)**   **Max Usable Hosts (2\^h − 2)**   **CIDR**
  ------------------- --------------------------------- ----------
  3                   6                                 /29
  4                   14                                /28
  5                   30                                /27
  6                   62                                /26
  7                   126                               /25

*Using the table above, determine the host bits, CIDR, and block size
for each department.*

  **Department**              **Hosts Needed**   **Host Bits (h)**   **CIDR**   **Block Size**
  --------------------------- ------------------ ------------------- ---------- ----------------
  Production / Staff          60                 6                   /26        64
  VOIP Phones                 25                 5                   /27        32
  Guest Wi-Fi                 12                 4                   /28        16
  Management (switches/APs)   6                  3                   /29        8

Part B --- VLSM Allocation (carving the /24)
============================================

*Starting block: 192.168.10.0/24. Allocate subnets LARGEST to SMALLEST
to avoid fragmentation --- each new subnet starts right after the
previous one ends. Fill in the table using your Part A results.*

  **Department**       **CIDR**   **Network**      **Broadcast**    **Usable Range**
  -------------------- ---------- ---------------- ---------------- -------------------------------
  Production / Staff   /26        192.168.10.0     192.168.10.63    192.168.10.1-192.168.10.62
  VOIP Phones          /27        192.168.10.64    192.168.10.95    192.168.10.65-192.168.10.94
  Guest Wi-Fi          /28        192.168.10.96    192.168.10.111   192.168.10.97-192.168.10.110
  Management           /29        192.168.10.112   192.168.10.119   192.168.10.113-192.168.10.118

*Show your work for at least the first allocation (Production/Staff) ---
the binary or block-boundary math you used to find its network and
broadcast address: /26=64 host /27=32 host /28=16 host /29=8 host*

64-1= 63 0-63

64+32-1=95 64-95

96+16-1=111 96-111

112+8-1=119 112-119

Part C --- Build & Verify
=========================

*Packet Tracer track: build a router (or Layer 3 switch) with four VLANs
matching your subnets. Assign one PC per VLAN using an address from your
allocated range. Ping within the same VLAN (should succeed) and across
VLANs without inter-VLAN routing configured (should fail). Screenshot
both results.*

*\[ Insert same-VLAN ping (success) screenshot here \]*
![](images/media/image1.png){width="7.0in"
height="6.969444444444444in"}

*\[ Insert cross-VLAN ping (fails without routing) screenshot here \]*
![Uploaded
image](images/media/image2.png){width="7.0in"
height="7.082638888888889in"}

*VM alternative: assign a secondary address from two different subnets
to your VM\'s interface (sudo ip addr add \<ip\>/\<cidr\> dev enp0s3),
then run ip route show and explain what the routing table tells you
about which subnets your VM can reach directly.*

\$ ip route show

(paste output)

Part D --- Access Control Reasoning
===================================

*For each pair of subnets, decide Allow or Deny for traffic flowing from
the row (source) to the column (destination). Think about which
departments have a legitimate business reason to reach each other.*

  **Source ↓ / Destination →**   **Production**   **VOIP**   **Guest**   **Management**
  ------------------------------ ---------------- ---------- ----------- ----------------
  Production                     Allow            Deny       Deny        Deny
  VOIP                           Allow            Allow      Deny        Deny
  Guest                          Deny             Deny       Allow       Deny
  Management                     Allow            Allow      Allow       Allow

*For each Deny you marked, justify it in one sentence citing a specific
security principle (e.g., least privilege, need-to-know, network
segmentation). At minimum, justify why Guest must be denied access to
every other subnet, and why Management should be reachable only from an
authorized admin source.*

Guest is denied everywhere because it has no legitimate role on the
network. Allowing it access would create an unauthorized attack path
into internal traffic. This segmentation prevents breaches or
unauthorized monitoring from a compromised guest device. Management is
allowed broad access because administrators need reach into every subnet
to do their job, applying least privilege in the opposite direction.
Everyone else only gets what they need, and only Management has that
authority.

Part E --- GRC Bridge: Subnet Allocation Policy
===============================================

1. Subnet Allocation Policy statement
-------------------------------------

*Draft a policy statement (4--6 sentences) for how this business manages
its subnets going forward. Must address: (a) how new subnets are sized
and requested, (b) where subnet assignments are documented, (c) the
default-deny stance between subnets unless explicitly approved, (d) who
has authority to approve exceptions.*

Subnets size will be based off how many end user workstations we have;
requests are sized to the actual number of hosts needed. Subnet
assignments will be documented in an internal network topology map.
Subnets cannot talk to each other by default, and communication is only
allowed if someone explicitly approves it. Least privilege will be in
effect, so each subnet is only granted access to what is needed.
Management or the Network administrator will decide who has access to
what.

2. Executive justification
--------------------------

*In 3--4 sentences, explain to a non-technical business owner why Guest
Wi-Fi must be on a completely separate subnet from Production/Staff,
using the same reasoning about broadcast domains and passive listening
you used in Lab 2.*

Think of WIFI and network segmentation as a house party. If you have
guests over, you will only want them in the living room and no access to
your bedrooms. Think of your living room being guests WIFI where any and
everyone is allowed in. The rooms are the isolated networks that only
you and family have access to. You wouldn't want guests coming into your
rooms or listening to your private conversations within your room. On a
shared network, every device can technically see traffic from other
devices passing by, so a guest\'s laptop could pick up business data the
same way someone in your living room could overhear a private
conversation happening a few rooms away.

Submission Checklist
====================

*Confirm each item is complete before considering Lab 3 done.*

-   Host-bits sizing table completed for all four departments (Part A)

-   VLSM allocation table with network/broadcast/usable range for all
    four subnets, plus shown work (Part B)

-   Same-VLAN and cross-VLAN screenshots, or VM routing table output
    with explanation (Part C)

-   Completed access control matrix with justifications for each Deny
    (Part D)

-   Subnet Allocation Policy statement and executive justification
    (Part E)
