# Enterprise Network Security and Access Control
**Week 04 — IT-Simplera Network Administration Internship Program**

Design and implementation of a secure enterprise network in GNS3, featuring SSH-based secure remote management, Standard/Extended/Named ACLs for departmental traffic control, and static routing across a two-switch, one-router topology.

![Topology](Screenshots/01_topology.png)

## 📌 Project Overview

This project simulates a small enterprise network with four departments (Management, Finance, HR, Development) and two shared services (Web-Server, DHCP-Server), connected through a central router (**R1**) and two Layer-3 access switches (**ESW1**, **ESW2**). The goal was to secure the network using:

- **Secure remote management** — SSHv2 with a 2048-bit RSA key pair, replacing Telnet
- **Access Control Lists** — Standard, Extended, and Named ACLs enforcing a departmental security policy
- **Static routing** — full reachability between all VLANs and shared services
- **Router-level name resolution** — local host tables for administrative convenience

📄 Full write-up: [`Docs/Week4_Enterprise_Network_Security_Report.docx`](Docs/Week4_Enterprise_Network_Security_Report.docx)

## 🗺️ Topology

| Device | Role | Key Interfaces |
|---|---|---|
| **R1** | Core router | Fa0/0 → ESW1 (192.168.12.1/30), Gi1/0 → ESW2 (192.168.23.1/30) |
| **ESW1** | L3 access switch | VLAN 10 (Management), VLAN 20 (Finance), VLAN 100 (Web-Server) |
| **ESW2** | L3 access switch | VLAN 30 (HR), VLAN 40 (Development), VLAN 100 (DHCP-Server) |

## 🧮 IP Addressing Plan

| Segment / VLAN | Network | Gateway | Device |
|---|---|---|---|
| Management (VLAN 10) | 10.1.10.0/24 | 10.1.10.1 | 10.1.10.50 |
| Finance (VLAN 20) | 10.1.20.0/24 | 10.1.20.1 | 10.1.20.50 |
| HR (VLAN 30) | 10.1.30.0/24 | 10.1.30.1 | 10.1.30.50 |
| Development (VLAN 40) | 10.1.40.0/24 | 10.1.40.1 | 10.1.40.50 |
| Shared Services (VLAN 100) | 10.1.100.0/24 | .1 (ESW1) / .2 (ESW2) | Web-Server 10.1.100.80 |
| R1 ↔ ESW1 | 192.168.12.0/30 | — | routed link |
| R1 ↔ ESW2 | 192.168.23.0/30 | — | routed link |

## 🔐 Security Implementation

### 1. Secure Remote Management (SSH)
Telnet was disabled; VTY lines accept SSHv2 only, backed by a 2048-bit RSA key pair and local authentication.
```
ip domain name local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
line vty 0 4
 transport input ssh
 login local
```
📷 [`07_ssh_device_hardening.png`](Screenshots/07_ssh_device_hardening.png) — `show ip ssh` verification
📷 [`08_domain_resolution_test.png`](Screenshots/08_domain_resolution_test.png) — name-resolution ping test

### 2. Access Control Lists

| ACL | Type | Purpose |
|---|---|---|
| `10` | Standard, numbered | Restrict VTY management access to the Management subnet only |
| `101` | Extended, numbered | Deny HR subnet HTTP/HTTPS access to the Web-Server; permit everything else |
| `SECURE_ZONE_POLICY` | Extended, **named** | Permit Management→Web-Server ICMP · deny Development→Finance · permit DHCP relay · permit the rest |

📷 [`09_show_ip_access_lists.png`](Screenshots/09_show_ip_access_lists.png) — live ACLs with match counters
📷 [`10_acl_icmp_prohibited.png`](Screenshots/10_acl_icmp_prohibited.png) — denied traffic returns **ICMP Type 3, Code 13** (administratively prohibited)
📷 [`11_acl_permitted_evidence.png`](Screenshots/11_acl_permitted_evidence.png) — permitted traffic flows normally

### 3. Verification & Testing

📷 [`02_ip_addressing_verification.png`](Screenshots/02_ip_addressing_verification.png) — client-side addressing check
📷 [`03_r1_host_table.png`](Screenshots/03_r1_host_table.png) — router-level `show hosts`
📷 [`12_end_to_end_reachability.png`](Screenshots/12_end_to_end_reachability.png) — Management → Web-Server gateway, 5/5 success

## 📁 Repository Structure

```
.
├── Configs/                 # Cisco IOS-style configuration exports
│   ├── R1.cfg
│   ├── ESW1.cfg
│   └── ESW2.cfg
├── Screenshots/             # Numbered verification evidence (see table above)
├── Docs/
│   ├── Week4_Enterprise_Network_Security_Report.docx   # Full project report
│   └── Week4_Assignment_Brief.pdf                       # Original assignment
└── README.md
```

## ✅ Results Summary

| Test | Expected | Result |
|---|---|---|
| Management PC addressing | Correct IP/gateway/DNS | ✅ Pass |
| Gateway reachability (Mgmt → Web-Server) | Permit | ✅ Pass — 5/5, 13–17 ms |
| Name resolution (`dhcp-server.local`) | Resolve & reply | ✅ Pass |
| Inter-VLAN restriction (Dev → Finance) | Deny | ✅ Pass — admin. prohibited |
| ACL match counters | Increment on denied traffic | ✅ Pass — 5 matches |
| SSH remote management | SSHv2 only, Telnet refused | ✅ Pass — RSA 2048-bit |

## 🛠️ Tools Used
GNS3 · SolarWinds Solar-PuTTY (SSH client) · VPCS

## 🚧 Known Gaps / Next Steps
Layer 2 hardening (Port Security, DHCP Snooping, BPDU Guard, Root Guard, PortFast) is part of the assignment scope but not yet captured with verification evidence in this iteration — tracked as a follow-up.

## 👤 Author
Network Administration Intern — IT-Simplera Institute
Supervised by Jawad Qayum (Senior Network Administrator)
