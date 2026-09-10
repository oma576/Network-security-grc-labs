**HACKING THE WORKFORCE**

**Lab 2 --- Logical Boundaries & IP Addressing**

*Week 2 · OSI Layer 3 · Worksheet & Submission*

+-------------+--------------+----------------------+----------------+
| **Name**    | **Date**     | **Environment used** | **Time spent** |
+-------------+--------------+----------------------+----------------+
| Omar Swoope | July 29 2026 | ACCESS sandbox /     | 3              |
|             |              |                      |                |
|             |              | Linux VM / Packet    |                |
|             |              |                      |                |
|             |              | Tracer               |                |
+-------------+--------------+----------------------+----------------+

Part A --- Number Systems (pencil first, calculator never)
==========================================================

1. Decimal → binary
-------------------

*Convert each decimal octet to 8-bit binary, showing your subtraction
work (place values: 128·64·32·16·8·4·2·1).*

+-------------+-----------------------------------------------+
| **Decimal** | **Binary (8 bits) --- show subtraction work** |
+-------------+-----------------------------------------------+
| 192         | 11000000                                      |
|             |                                               |
|             | 128+64=192                                    |
+-------------+-----------------------------------------------+
| 168         | 10101000                                      |
|             |                                               |
|             | 128+32+8=168                                  |
+-------------+-----------------------------------------------+
| 10          | 00001010                                      |
|             |                                               |
|             | 8+2=10                                        |
+-------------+-----------------------------------------------+
| 172         | 10101100                                      |
|             |                                               |
|             | 128+32+8+4=172                                |
+-------------+-----------------------------------------------+
| 255         | 11111111                                      |
|             |                                               |
|             | 128+64+32+16+8+4+2+1=255                      |
+-------------+-----------------------------------------------+

2. Binary → decimal
-------------------

*Convert each binary octet back to decimal.*

  ------------ -------------
  **Binary**   **Decimal**
  11000000     192
  10101100     172
  11100000     224
  01111111     127
  ------------ -------------

3. Hex conversion + reasoning
-----------------------------

*Convert the first two byte-pairs of your Lab 1 MAC address into binary,
then back into hex, showing the 4-bit groupings.*

08:00

08=0000 1000 00=0000 0000

0000 1000= 08 0000 0000=00

*Why does networking use hex for MACs but dotted decimal for IPv4?*

Hex is used for MAC because its address breaks down into 12 hex digits
(4 bits each) and easy to convert by hand. IPv4 addresses are constantly
used for arithmetic whether it's subnetting, calculating ranges, and
figuring out network/broadcast. Dotted decimal fits that kind of math
more than hex.

Part B --- Unicast, Multicast, Broadcast (worksheet)
====================================================

1. Address classification
-------------------------

*Classify each address as unicast, multicast, directed broadcast,
limited broadcast, or Layer 2 broadcast (assume 192.168.1.0/24 where
relevant), and note who receives traffic sent to it.*

  ------------------- -------------------- -------------------------------------------------------
  **Address**         **Type**             **Who Receives It?**
  192.168.1.45        Unicast              Only one device with that address
  224.0.0.251         Multicast            Only devices subscribed to that multicast group
  192.168.1.255       Directed broadcast   Every device on the 192.168.1.0/24 subnet receives it
  255.255.255.255     Limited broadcast    Every device on the local network segment
  ff:ff:ff:ff:ff:ff   Layer 2 broadcast    Every device on the local network segment
  ------------------- -------------------- -------------------------------------------------------

2. tcpdump observation
----------------------

*Run: sudo tcpdump -i enp0s3 -c 10 \'broadcast or multicast\' --- let it
run for a minute on a busy segment.*

\$ sudo tcpdump -i enp0s3 -c 10 \'broadcast or multicast\'

(paste output) 15:05:58.576774 IP6 \_gateway \> ip6-allnodes: ICMP6,
router advertisement, length 80

15:05:58.580504 IP6 workforcelab \> ff02::16: HBH ICMP6, multicast
listener report v2, 1 group record(s), length 28

15:05:59.812925 IP6 workforcelab \> ff02::16: HBH ICMP6, multicast
listener report v2, 1 group record(s), length 28

*List two protocols you observed and what a passive listener could learn
about the network from them without sending a single packet. ICMP6*

ICMPv6, and MLDv2 are the protocols I observed. A passive listener could
learn the identity of the network's router, and that specific devices
exist on the network and observe its multicast activity. The router
sends a message announcing itself and the networks address to all IPv6
devices. MLDv2 messages were sent by my VM announcing which multicast
groups it wants to receive traffic.

Part C --- Netwothrk & Broadcast Addresses (pencil + verification)
==================================================================

1. Subnet calculations
----------------------

*For each IP/mask pair, calculate the network address, broadcast
address, and usable host range. Show the binary AND for at least the
first row (attach separately or in the space below the table).*

  ------------------- --------------- --------------- -----------------------------
  **IP / Mask**       **Network**     **Broadcast**   **Usable Range**
  172.16.54.29 /24    172.16.54.0     172.16.54.255   172.16.54.1-172.16.54.254
  10.0.10.77 /26      10.0.10.64      10.0.10.127     10.0.10.65-10.0.10.126
  192.168.1.200 /27   192.168.1.192   192.168.1.223   192.168.1.193-192.168.1.222
  10.0.10.130 /25     10.0.10.128     10.0.10.255     10.0.10.129-10.0.10.254
  ------------------- --------------- --------------- -----------------------------

2. The \"aha\" exercise
-----------------------

*Host A is 10.0.10.128 and Host B is 10.0.10.64. With mask 255.255.255.0
they communicate normally. The mask on both is changed to
255.255.255.128. Using your row-4 math, explain precisely why they can
no longer reach each other directly, and what device would now be
required between them.*

They can no longer communicate because those hosts now operate on
different subnets. A router is needed to connect them.

3. Sandbox verification
-----------------------

*Verify one row in your VM: sudo ip addr add 10.0.10.77/26 dev enp0s3
(or a secondary interface), then ip addr show and confirm the computed
broadcast matches your hand calculation.*

*\[ Insert verification screenshot here
\]*![](images/media/image1.png){width="7.0in"
height="4.892361111111111in"}

Part D --- Incident Scenario: Duplicate IP Conflict
===================================================

*THE TICKET: A case manager\'s workstation (192.168.1.50) keeps dropping
off the network \"every few minutes, then it comes back.\" Yesterday a
volunteer set up a donated printer somewhere in the office. Nothing else
has changed.*

1. Hypothesis
-------------

*Before touching the keyboard: what do the symptoms (intermittent,
alternating connectivity) suggest, and why does the printer detail
matter?*

I'm thinking the printer is disrupting the signal causing the
workstation to keep dropping off.

2. ARP evidence
---------------

*From a third machine on the same segment: ip neigh show \| grep
192.168.1.50 --- run it, then ping the host and check again. Paste both
results.*

\$ ip neigh show \| grep 192.168.1.50

(paste output) 192.168.1.50 dev eth0 lladdr a4:5e:60:b2:11:09 REACHABLE

\$ ping -c 2 192.168.1.50 && ip neigh show \| grep 192.168.1.50

(paste output) 192.168.1.50 dev eth0 lladdr 00:1b:a9:c3:77:d2 REACHABLE

*What does it mean that the same IP resolves to two different MAC
addresses at different moments?*

It seems like the same IP address is assigned to two different devices
causing the disruption thus why two different physical network addresses
is showing. This causes the network to intermittently route traffic to
whichever device most recently responded to an ARP request.

3. Active duplicate-address probe
---------------------------------

*sudo arping -D -c 4 -I enp0s3 192.168.1.50 --- paste the result
confirming DUPLICATE ADDRESS DETECTED.*

\$ sudo arping -D -c 4 -I enp0s3 192.168.1.50

(paste output) Unicast reply from 192.168.1.50 \[a4:5e:60:b2:11:09\]
Unicast reply from 192.168.1.50 \[00:1b:a9:c3:77:d2\] \... 2 replies ---
DUPLICATE ADDRESS DETECTED

4. Root cause and remediation
-----------------------------

*Using the OUI skill from Lab 1, identify which of the two MACs likely
belongs to the printer. Document root cause and your remediation steps
(immediate fix and the configuration change that prevents recurrence).*

00:1b:a9:c3:77:d2 belongs to the printer. I determined the root cause IP
address is assigned to two different devices or MACs, the printer should
be reassigned to another IP. Move the network to DHCP for automated IP
assignment, to remove the possibility of two devices colliding on the
same address again.

5. Policy question
------------------

*The volunteer assigned the printer a static IP \"because that\'s how
the old office did it.\" What organizational controls would have
prevented this incident? Name at least two.*

Only IT personnel should be allowed to connect devices to the network.
IT should also keep a documented list of assigned IP addresses so no one
hands out an address that\'s already in use.

Part E --- GRC Bridge: Network Isolation Policy
===============================================

1. Network Isolation policy statement
-------------------------------------

*Draft a policy statement (4--6 sentences) for a nonprofit that handles
client health records. Must address: (a) why HR/client-records systems
and guest Wi-Fi must never share a broadcast domain, (b) how IP
addresses are assigned and by whom, (c) how exceptions are requested and
approved.*

Guest Wi-Fi and staff/client records systems must be on two different
networks so a guest device can never observe or reach systems holding
client data. Staff workstations will be on a separate VLAN, isolated
from the guest network\'s broadcast domain, protecting client data from
exposure. IP address assignment will be automated by DHCP, managed only
by IT staff within the organization. Any request to change IP
configuration must go through IT and must be documented.

2. Executive explanation
------------------------

*In two or three sentences, explain to a non-technical executive
director how the multicast/broadcast traffic you captured in Part B
demonstrates why isolation protects client privacy.*

Imagine walking into a room full of random people and announcing you
have confidential information, now everyone in that room knows you\'re
there and can listen for anything useful you say. That\'s essentially
what happens on a shared network. Devices constantly announce
information about themselves, and anyone sharing that same network can
hear it. Keeping guest Wi-Fi on a separate network means visitors are
never in the same \'room\' as the systems holding client data in the
first place.

Submission Checklist
====================

*Confirm each item is complete before submitting Lab 2.*

-   Both conversion tables with visible work, plus the hex question
    (Part A)

-   Address classification table and tcpdump observations (Part B)

-   All four subnet calculations, the \"aha\" explanation, and the
    verification screenshot (Part C)

-   Complete incident write-up: hypothesis, evidence, root cause,
    remediation, and controls (Part D)

-   Network Isolation policy statement and executive explanation
    (Part E)
