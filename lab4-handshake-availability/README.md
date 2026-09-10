**HACKING THE WORKFORCE**

**Lab 4 --- The Handshake & Availability**

*Week 4 · OSI Layer 4 (TCP) · Worksheet & Submission*

*Self-directed edition --- built from the program\'s Week 4 preview,
since full instructions weren\'t available outside the cohort platform.*

  ------------- ---------------- ---------------------- ----------------
  **Name**      **Date**         **Environment used**   **Time spent**
  Omar swoope   August 12 2026   vm                     3
  ------------- ---------------- ---------------------- ----------------

Scenario
========

*The IT team for a small business wants to confirm two things: that
connections to its servers are establishing properly (and can be trusted
to explain when they don\'t), and that no unauthorized "shadow IT"
services are quietly listening on company machines. You\'ll capture a
real TCP handshake, audit your VM\'s open ports like an auditor would,
trace how your private traffic reaches the public internet through NAT,
and write the policy that governs which services are allowed to run at
all.*

Part A --- The TCP Three-Way Handshake
======================================

1. Concept, in your own words
-----------------------------

*Explain what SYN, SYN-ACK, and ACK each mean, and why a connection
needs all three before data starts flowing. Fill in the table, then
explain below.*

  ---------- ------------ ----------------- ----------------------------------------------------------------------------------------------------
  **Step**   **Sender**   **Flag(s) Set**   **Purpose**
  1          client       SYN               Client requests to open a connection
  2          server       SYN-ACK           Server acknowledges request and confirms it's willing to connect too.
  ~3~        ~client~     ~ACK~             ~Client\ confirms\ receipt\ of\ the\ server's\ SYN-ACK.\ Connection\ is\ now\ fully\ established.~
  ---------- ------------ ----------------- ----------------------------------------------------------------------------------------------------

All three steps must happen for data to be sent from each side. Think of
this process as a conversation both sides must hear each other for it to
start.

2. Capture it live
------------------

*On your VM, start a capture, then generate traffic to trigger a
handshake:*

\$ sudo tcpdump -i \<interface\> -n \'tcp\[tcpflags\] &
(tcp-syn\|tcp-ack) != 0\' &

\$ curl -v http://example.com (or: ssh to another host / your own VM)

*Identify the SYN, SYN-ACK, and ACK lines in your capture output and
screenshot them.*

*\[ Insert tcpdump capture showing the 3-way handshake here \]* ![Full
size
preview](images/media/image1.png){width="7.0in"
height="4.8597222222222225in"}

*What would you expect to see in the capture if the final ACK never
arrived (e.g., dropped by a firewall)? What does that mean for the
connection?*

The connection would never fully establish. The server would be waiting
to hear back for a connection that never completes. In the capture,
you\'d still see the SYN and SYN-ACK, since those already happened, but
the client\'s final ACK would simply be missing. The server eventually
times out and drops the connection attempt.

Part B --- Shadow IT Hunt (ss + nmap)
=====================================

*"Shadow IT" means software or services running that IT never approved
--- every open port is a door into the machine, so an auditor\'s job is
to find every door and confirm it\'s supposed to be open.*

1. Inside view --- ss
---------------------

\$ sudo ss -tulnp

*List every listening port ss reports, and screenshot the output.*

*\[ Insert ss -tulnp output here
\]*![](images/media/image2.png){width="7.0in"
height="4.822916666666667in"}

2. Outside view --- nmap
------------------------

\$ sudo apt install -y nmap

\$ nmap -sT -p- \<your VM\'s IP\>

*Scan your own VM from the network\'s perspective. Screenshot the
results.*

*\[ Insert nmap scan output here \]* ![Full size
preview](images/media/image3.png){width="7.0in"
height="4.821527777777778in"}

3. Reconcile the two views
--------------------------

*Fill in the inventory table: every port either tool reported, whether
it\'s expected, and what you\'d do about anything unexpected (leave it,
investigate it, or shut it down).*

  ---------- -------------- ---------------------------------------- --------------------- ------------------------------------------------------
  **Port**   **Protocol**   **Process / Service**                    **Expected? (Y/N)**   **Action / Risk Note**
  53         UDP            DNS Resolves hostnames to IP addresses   Y                     Leave it
  68         UDP            DHCP                                     Y                     Leave it
  53         TCP            systemd-resolve (DNS stub resolver)      Y                     Leave it --- loopback only, not reachable externally
                                                                                           
                                                                                           
  ---------- -------------- ---------------------------------------- --------------------- ------------------------------------------------------

*Why is it useful to check both the inside view (ss, from the host
itself) and the outside view (nmap, from the network)? What could one
catch that the other would miss?*

Using only ss is like having one point of view. Sophisticated malware
could hide itself from ss, but nmap looks at ports from outside the
machine and could still catch it. At the same time, ss and nmap cover
each other\'s blind spots. ss caught the DNS/DHCP services running on
loopback, but nmap couldn\'t see those at all since they\'re not
reachable from the network.

Part C --- Tracing NAT
======================

*Your VM has a private address that no device on the public internet can
route to directly --- NAT is what lets it reach the internet anyway.*

\$ ip addr show (note your VM\'s private IP)

\$ curl ifconfig.me (or curl https://api.ipify.org)

*Record both addresses and screenshot the output.*

*\[ Insert ip addr + curl ifconfig.me output here \]* ![Full size
preview](images/media/image4.png){width="7.0in"
height="4.905555555555556in"}

+----------------------------------------------------+
| **Recorded addresses**                             |
|                                                    |
| VM private IP: 10.0.2.15/24                        |
|                                                    |
| Public IP (as seen by the internet): 208.54.99.213 |
+----------------------------------------------------+

*Explain, in your own words, why these two addresses are different, and
what NAT is doing to your traffic (packets and source ports) as it
leaves your network. Why couldn\'t a device on the internet just send
traffic straight to your VM\'s private IP?*

These two addresses are different because NAT translates your private IP
into a public one when connecting to the internet. Your private IP is
non-routable, so it can only reach the internet by going through a
public IP via NAT. NAT also rewrites the source port on each packet. If
multiple devices share one public IP, the router uses the port number to
remember which device to send each reply to.

Part D --- GRC Bridge: Service Hardening Standard
=================================================

1. Service Hardening Standard statement
---------------------------------------

*Draft a policy statement (4--6 sentences) for how this business decides
which services are allowed to run on its machines. Must address: (a) the
default posture for new/unknown services (allow or deny by default), (b)
who approves a service being turned on, (c) how open ports get audited
going forward and how often, (d) what happens when an unapproved service
is found.*

To harden our systems, new services will be denied unless approved by
the network admin. A new service must be evaluated for how it could
affect our systems, weighing the pros and cons before approval. ss and
nmap will be run every day to audit open ports. If any unapproved
services are found, new services will be disabled until formally
reviewed.

2. Executive justification
--------------------------

*In 3--4 sentences, explain to a non-technical business owner why an
"open port nobody remembers enabling" is a real business risk ---
connect it back to what you found (or could have found) in Part B.*

Think of ports as entrances into an office building, and security as the
network admin. If your entrances (ports) are unlocked, unauthorized
individuals can just walk in and blend in with the people who are
authorized. You need security (the network admin) to watch those
entrances, deny access to unauthorized people, and grant access to
authorized people. ss and nmap are the tools a network admin uses to
check if any services or unauthorized people aren\'t supposed to be
here.

Submission Checklist
====================

*Confirm each item is complete before considering Lab 4 done.*

-   Handshake table completed and explained in your own words (Part A.1)

-   tcpdump capture screenshot showing SYN / SYN-ACK / ACK (Part A.2)

-   ss -tulnp and nmap screenshots, plus reconciled service inventory
    table (Part B)

-   Private vs. public IP recorded and NAT explained (Part C)

-   Service Hardening Standard statement and executive justification
    (Part D)
