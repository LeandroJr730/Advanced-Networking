# Requirements & Tech Stack

[← Back to README](README.md)

---

## ✅ Project Rubric

| Requirement | Points | Status |
|---|---|---|
| Ping outside router/firewall from inside LAN | 10 | ✅ |
| Each workstation on a separate VLAN, communicating within LAN | 10 | ✅ |
| Workstations joined to the domain | 10 | ✅ |
| DHCP on Domain Controller providing addresses | 10 | ✅ |
| Internal DNS resolving the mail server | 10 | ✅ |
| Send mail internally within the LAN | 10 | ✅ |
| Send mail externally to other group | 10 | ✅ |
| Wireless access via WAP | 10 | ✅ |
| ACL rule blocking port 80 from outside | 10 | ✅ |
| Completed documentation | 20 | ✅ |
| Participation | 40 | ✅ |
| **Total** | **150** | |

---

## 🛠️ Hardware

| Device | Role |
|---|---|
| 2× Dell PowerEdge rack-mounted servers | Server1 (DC) and Server2 (Email) |
| 2x Dell Precision Tower workstation | Workstations |
| Cisco router | Inter-VLAN routing & external link |
| HP managed switch | VLAN segmentation |
| Wireless Access Point (WAP) | Wireless client access |
| KVM switch | Server management |

---

## 💻 Software & Services

| Software | Purpose |
|---|---|
| Windows Server 2022 | Server OS (bare-metal) |
| Windows 11 Pro | Workstation OS |
| Active Directory Domain Services | Domain controller (`south.com`) |
| DNS & DHCP | Name resolution and IP management |
| hMailServer | Email server (SMTP/IMAP/POP3) |
| Mozilla Thunderbird | Email client |
| Pale Moon | Legacy WAP GUI access |

---

## 🔧 Networking Concepts Applied

- VLANs (access & trunk ports)
- Router-on-a-Stick (sub-interfaces)
- DHCP Relay (`ip helper-address`)
- DNS A & MX records
- Static routing
- Access Control Lists (blocking port 80)
- Subnetting (/26 per VLAN)
- WAP configuration and wireless integration
