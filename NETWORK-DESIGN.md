# Network Design

[← Back to README](README.md)

---

## 🗺️ Topology

**Internal IP Network:** `192.168.13.0/24`
**Domain:** `south.com`

The network uses a **router-on-a-stick** architecture with one Cisco router handling inter-VLAN routing through sub-interfaces, one managed HP switch segmenting traffic into three VLANs, two physical servers, two workstations, and one WAP.

---

## VLAN Plan

| VLAN | Purpose | Subnet | IP Range | Gateway |
|---|---|---|---|---|
| VLAN 10 | Workstation 1 | 192.168.13.0/26 | .1 – .62 | 192.168.13.1 |
| VLAN 20 | Workstation 2 | 192.168.13.64/26 | .65 – .126 | 192.168.13.65 |
| VLAN 30 | Servers & WAP | 192.168.13.128/26 | .129 – .190 | 192.168.13.129 |

---

## IP Assignments

| Device | Role | IP Address |
|---|---|---|
| Server1 | Domain Controller, DHCP, DNS | 192.168.13.130 |
| Server2 | Email Server (hMailServer) | 192.168.13.131 |
| Router sub-interface .10 | VLAN 10 gateway | 192.168.13.1 |
| Router sub-interface .20 | VLAN 20 gateway | 192.168.13.65 |
| Router sub-interface .30 | VLAN 30 gateway | 192.168.13.129 |
| Router outside interface | Inter-group link | 192.168.6.1 |
| Group 2 router | Static route target | 192.168.6.7 |
| DHCP scope | Dynamic leases | 192.168.13.50 – .100 |

---

## DNS Records (on Server1)

| Record | Type | Points To |
|---|---|---|
| mail.south.com | A | 192.168.13.131 |
| south.com | MX | mail.south.com |
| mail.north.com | A | *(Group 2's mail server IP)* |
