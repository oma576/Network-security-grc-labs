# Network Security & GRC Labs — Hacking the Workforce (Self-Directed)

This repo documents five hands-on labs I completed while working through the *Hacking the Workforce* network security / GRC pre-apprenticeship curriculum on my own — without access to the program's official Zoom sessions or premium sandbox platform. I built my own lab environment (VirtualBox + Ubuntu Server, Cisco Packet Tracer) and, for the labs where the program's public materials were limited to a short topic preview, designed my own equivalent practical exercises covering the same learning objectives.

Each lab folder contains my full worksheet — write-ups, terminal/Packet Tracer screenshots, and the policy documents I drafted — converted to Markdown for easy reading here on GitHub.

## Environment

- **VM:** Ubuntu Server 24.04 LTS running in VirtualBox
- **Network simulation:** Cisco Packet Tracer (VLANs, switch CLI, inter-VLAN routing tests)
- **Tools used across labs:** `ip`, `ss`, `nmap`, `tcpdump`, `arping`, `curl`, `ethtool`

## Labs

| Lab | Topic | Key Skills |
|---|---|---|
| [Lab 1 — Physical & Data Link](lab1-physical-datalink/) | Cabling, MAC addresses, OUI lookup, frame health | Cat 6/6a cabling standards, MAC address structure, `ethtool` error counters |
| [Lab 2 — IP Addressing](lab2-ip-addressing/) | Number systems, unicast/multicast/broadcast, subnetting, incident response | Binary/hex conversion, subnet math, duplicate-IP conflict investigation with `arping`/ARP |
| [Lab 3 — Subnetting Puzzle & Access Control](lab3-vlsm-vlans-access-control/) | VLSM, VLAN segmentation, access control policy | Variable-length subnet masking, VLAN design in Packet Tracer, access control matrix, subnet allocation policy |
| [Lab 4 — The Handshake & Availability](lab4-handshake-availability/) | TCP handshake, shadow IT hunting, NAT | Packet capture with `tcpdump`, open-port auditing (`ss` + `nmap`), NAT tracing, service hardening policy |
| [Lab 5 — Discovery, Routing & Compliance](lab5-discovery-routing-compliance/) | Routing tables, ARP, neighbor discovery, capstone audit | Routing table analysis, ARP spoofing risk, CDP/LLDP exposure, final network audit report, incident response policy |

## Reference

[Networking Quick Reference](reference/) — a study sheet I built covering binary/hex conversion and subnetting fundamentals.

## Why self-directed labs for 3–5?

The public preview material for weeks 3–5 was limited to a one-paragraph topic description per lab, since full instructions and answer keys are gated behind the program's premium platform. Rather than wait, I built my own practical exercises covering the same stated learning objectives (VLSM/VLANs/access control; TCP/NAT/service hardening; routing/ARP/discovery/compliance), scoped to match the depth of the labs before them.
