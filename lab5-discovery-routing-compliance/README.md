**HACKING THE WORKFORCE**

**Lab 5 --- Discovery, Routing & Compliance (Capstone)**

*Week 5 · Routing · ARP · Neighbor Discovery · Worksheet & Submission*

*Self-directed edition --- built from the program\'s Week 5 preview,
since full instructions weren\'t available outside the cohort platform.*

  ------------- ---------------- ---------------------- ----------------
  **Name**      **Date**         **Environment used**   **Time spent**
  Omar Swoope   August 15 2026   VM                     3
  ------------- ---------------- ---------------------- ----------------

Scenario
========

*This is the capstone lab. The business wants a full audit of its
network, confirmation that devices can be identified and trusted, and a
documented plan for how it will detect and respond to incidents going
forward. You\'ll read your VM\'s routing table, watch ARP resolve IP
addresses to physical MAC addresses, use neighbor discovery to see what
a switch can learn about connected devices, then pull everything from
Labs 1--4 together into one audit report and the policy that governs how
incidents get handled.*

Part A --- Reading the Routing Table
====================================

1. Concept
----------

*Every device keeps a routing table that answers one question for every
outgoing packet: "can I deliver this directly, or does it need to go
through a gateway?" Explain in your own words what a default gateway is,
and why a device needs a routing table at all instead of just sending
every packet straight to its destination.*

A default gateway is a router connecting its LAN to other networks or
the internet. The routing table is needed to decide whether packets are
being sent to its subnet or another. Without the table, the device
couldn\'t do either.

2. Read your VM\'s table
------------------------

\$ ip route show

*Screenshot the output, then fill in the table below: your default
route, and your local subnet route.*

*\[ Insert ip route show output here \]* ![Full size
preview](images/media/image1.png){width="7.0in"
height="4.910416666666666in"}

  ----------------- ------------------- --------------- -----------------------------------------
  **Destination**   **Gateway / Dev**   **Interface**   **Purpose**
  Default           10.0.2.2            Enp0s3          Reach anything outside my subnet
  10.0.2.0/24       No gateway needed   Enp0s3          Reach devices on my own subnet directly
  ----------------- ------------------- --------------- -----------------------------------------

*If the default route entry didn\'t exist, could your VM still reach
something on its own subnet? Could it reach example.com? Explain why,
connecting it back to what "default gateway" means.*

Yes, the VM could still reach devices within its own subnet without a
default route entry. But it couldn\'t reach example.com because it\'s
the default gateway\'s job to connect its network to outside networks or
the internet. This connects back to what a default gateway means: it\'s
the route that connects to devices outside its subnet.

Part B --- ARP: Mapping IP to MAC
=================================

1. Concept, in your own words
-----------------------------

*ARP (Address Resolution Protocol) is how a device finds the MAC address
that belongs to an IP address on its local network. Fill in the table
for the two-step exchange, then explain below.*

  ---------- --------------- ------------------ -----------------------------------------------------------
  **Step**   **Sender**      **Message Type**   **Purpose**
  1          VM              broadcast          Ask which device on the subnet owns a specific IP address
  2          Target device   unicast            Replies with MAC address
  ---------- --------------- ------------------ -----------------------------------------------------------

*Why does the first message have to be a broadcast, instead of being
sent directly to one device?*

Because the VM knows the target's IP address but not its MAC address,
and ethernet delivery requires a MAC address. Since there's no way to
reach just that one device yet, the message must go to everyone on the
subnet.

2. Capture it live
------------------

*Clear your ARP cache and watch a fresh resolution happen:*

\$ ip neigh show

\$ sudo ip neigh flush all

\$ sudo tcpdump -i enp0s3 -n arp &

\$ ping -c 2 \<your default gateway IP\>

*Screenshot the tcpdump output showing the ARP request (who-has) and
reply (is-at), and the ip neigh show table populated afterward.*

*\[ Insert ARP c* ![Uploaded
image](images/media/image2.png){width="7.0in"
height="4.872222222222222in"} *apture + ip neigh show output here
\]*![](images/media/image3.png){width="7.0in"
height="4.937498906386701in"}

3. Security angle
-----------------

*ARP has no built-in authentication --- any device can claim to own any
IP address, and others will believe it. This is called ARP spoofing (or
ARP cache poisoning). Explain, in the simplest terms, how an attacker
could use this to intercept traffic between two other devices on the
same network. Connect it back to the duplicate-IP incident you
investigated in Lab 2 --- what\'s similar about the risk?*

An attacker could forge ARP replies to associate it's MAC address with
another device. Now the attacker can trick devices or two parties into
thinking their communicating with each other. So instead of talking
directly to each other, both devices unknowingly send their traffic
through the attacker first. This is like the duplicate-IP incident in
lab 2, where a device claiming an identity that wasn't really, theirs
caused confusion on the network. ARP spoofing is the same just with a
MAC address instead of an IP.

Part C --- Neighbor Discovery (CDP / LLDP)
==========================================

*Cisco Discovery Protocol (CDP) and its open-standard cousin LLDP let a
switch automatically learn what\'s plugged into each of its ports ---
device type, IP address, capabilities. Using your Lab 3 Packet Tracer
topology (or a fresh switch + 2 devices), run:*

Switch\# show cdp neighbors detail

*Screenshot the output.*

*\[ Insert show cdp neighbors detail output here
\]*![](images/media/image4.png){width="7.0in"
height="7.029861111111111in"}

*List two pieces of information CDP revealed about the neighboring
device. Then answer: this same information would be just as visible to
an attacker who plugged a laptop into an unused wall port. Should
CDP/LLDP be left enabled on every port, or only on trusted uplinks
between switches? Justify your answer using a security principle from an
earlier lab.*

The device ID and Interface are two pieces of info shown. This is a
vulnerability and should only be enabled for switch-to-switch uplinks.
All unused ports should be restricted as stated in previous labs, and
any changes must be approved and done by admin. This follows least
privilege, a port with no legitimate switch or router connected has no
need to broadcast this information.

Part D --- Final Network Audit Report
=====================================

*Pull together what you found across Labs 1--4 into one summary. This is
the kind of one-page report an auditor would hand to leadership.*

  ------------------------------------ ----------------------------------------------------------------------------------------------------------------- ------------------- ---------------
  **Category**                         **Finding**                                                                                                       **Status / Risk**   **Reference**
  Physical / Cabling                   Cat 6a is the most efficient and long-lasting cabling                                                             Low risk            Lab 1
  IP Addressing & Subnetting           Each department subnet is allocated from largest to smallest                                                      Low risk            Lab 3
  VLAN Segmentation & Access Control   Guest is denied on network only management has broad access                                                       Low risk            Lab 3
  Exposed Services (open ports)        No unexpected open ports found (ss + nmap both clean)                                                             Low risk            Lab 4
  NAT / External Exposure              VM's private IP is non-routable; NAT only translates outbound traffic, and no external reachable services exist   Low risk            Lab 4
  Neighbor Discovery Exposure          CDP reveals device details with no authentication                                                                 Moderate risk       Lab 5
  ------------------------------------ ----------------------------------------------------------------------------------------------------------------- ------------------- ---------------

*In 3--5 sentences, write your top 3 recommendations for this business
based on everything you found across all five labs.*

The business should implement port security to restrict MAC addresses on
unused ports, since our audit found CDP broadcasting device details on
any open port. The guest network should be isolated from the internal
LAN to prevent unauthorized traffic monitoring. ss and nmap audits
should be run on a consistent schedule; I suggest running them every day
to catch unauthorized services early.

Part E --- GRC Bridge: Incident Response & Visibility Policy
============================================================

1. Incident Response & Visibility Policy statement
--------------------------------------------------

*Draft a policy statement (4--6 sentences) for how this business detects
and responds to incidents going forward. Must address: (a) what
tools/checks are run regularly to maintain visibility (tie back to ss,
nmap, ARP, and CDP from this lab), (b) how a suspected incident --- a
rogue device, a duplicate IP, an unexpected open port --- gets reported,
(c) who has the authority to isolate or disconnect a suspect device from
the network, (d) how incidents get documented after they\'re resolved.*

Moving forward, ss and nmap will be used to check for open ports, ARP
checks will be run to catch forged replies, and CDP will be reviewed to
catch information leaking out of unused ports, all on a consistent
schedule. Whoever performs these checks and finds something unexpected
will report it directly to the network admin for investigation. Only the
network admin has authority to isolate or disconnect a suspicious device
from the network. The network admin will then document the root cause,
actions taken, and resolution for future reference.

2. Executive justification
--------------------------

*In 3--4 sentences, explain to a non-technical business owner why
"visibility" --- always knowing what devices are on the network and what
they\'re doing --- is the foundation that makes incident response
possible at all.*

Imagine you owned over 50 storage units that you rent out to paying
customers. The units are your ports and the customers are the devices.
You wouldn\'t want customers (attackers) to sneak in and use units
without paying. ss and nmap would be the security who does routine
checks on every unit for any freeloaders. Without those routine checks,
you wouldn\'t even know a freeloader was there until real damage was
already done, you can\'t respond to a threat you don\'t know exists.

Submission Checklist
====================

*Confirm each item is complete before considering Lab 5 --- and the
program --- done.*

-   Routing table concept explained and ip route show table completed
    (Part A)

-   ARP concept table, capture screenshot, and spoofing explanation
    (Part B)

-   CDP/LLDP screenshot and enabled-by-default justification (Part C)

-   Final audit report table and top-3 recommendations (Part D)

-   Incident Response & Visibility Policy and executive justification
    (Part E)
