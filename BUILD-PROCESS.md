# Build Process & Troubleshooting

[← Back to README](README.md)

---

## Phase 1 — Planning & Physical Setup

Designed the initial network diagram including device roles, IP addressing, VLANs, and subnetting. Received physical hardware, installed it in server racks, ran cables, reset workstations, and installed Windows 11 on both workstations.

---

## Phase 2 — Switch & Basic Connectivity

Connected via console cable to configure the switch. The switch hadn't been properly reset from a previous year — had to swap it out for a new one. Configured static IPs on all devices and verified basic ping connectivity.

---

## Phase 3 — VM Attempt (and why we abandoned it)

The original plan was to run two VMs on Server1: one for AD/DNS/DHCP and one for the email server, using Hyper-V. This approach ran into several problems:

- **BSODs on VM creation** — tried different ISOs, configurations, and creation methods. Switched to VMware Workstation Pro, which got the VMs running.
- **Bridged NIC issue** — VMs were configured as bridged but the NIC selection was set to automatic. Pings were failing until we manually pinned the adapter to the host server's NIC in VMware settings.
- **Duplicate SID** — VM2 was cloned from VM1, which caused a duplicate Security Identifier (SID). This prevented VM2 from joining the domain. Fixed using Sysprep to reset the SID.
- **Host server instability** — Both groups' servers were randomly shutting down throughout the lab. The root cause was never fully identified. After consulting the instructor, we decided to move away from VMs entirely to avoid compounding the existing Hyper-V issues and reduce risk going forward.

At this point we had working VMs with AD, DNS, DHCP, and a partial email setup — but the platform wasn't reliable enough to continue on.

---

## Phase 4 — Pivot to Bare-Metal

After the VM issues, we switched to bare-metal installations:

- Removed VMware and all VM-related files from Server1.
- Connected a KVM to both servers and booted Server2 from a USB with the Windows Server 2022 ISO, installing it directly on the hardware.
- Changed BIOS boot mode from Legacy to UEFI on both servers.

With both servers running bare-metal Windows Server 2022, we redid the core configurations. The setup was essentially the same as what we had done in the VMs — the main difference was that it was now stable.

---

## Phase 5 — Domain Controller, DNS & DHCP

Promoted **Server1** to Domain Controller and created the `south.com` forest. On Server1's DNS, added:
- An **A record** for `mail.south.com → 192.168.13.131`
- An **MX record** pointing `south.com` mail to `mail.south.com`

Configured an initial DHCP scope on Server1 for basic connectivity testing. This was later replaced with three separate VLAN scopes in Phase 8.

Joined both workstations to the `south.com` domain and verified DHCP leases.

> **Issue:** Workstations prompted for a password reset on first login even though no password had been set. Resolved through troubleshooting the default domain policy.

---

## Phase 6 — Email Server (hMailServer)

Installed hMailServer on **Server2** to handle SMTP, IMAP, and POP3.

> **Issue:** The installer required .NET Framework 2.0, which Windows Server 2022 couldn't install through normal means. Fixed by mounting a Windows Server ISO and running:
> ```powershell
> Install-WindowsFeature Net-Framework-Core -source D:\sources\sxs
> ```

Configured hMailServer with the `south.com` domain, created two user accounts (`leandro@south.com`, `steven@south.com`), enabled SMTP (25), IMAP (143), and POP3 (110), and added inbound firewall rules for those ports.

Installed Mozilla Thunderbird on both workstations. Configured incoming server as `mail.south.com` on IMAP port 143 and outgoing on SMTP port 25.

✅ Successfully sent an internal email from `leandro@south.com` to `steven@south.com`.

---

## Phase 7 — WAP Configuration

The WAP's CLI was unsupported, and the web GUI required SSL too old for modern browsers.

> **Issue:** Chrome, Edge (with security disabled), and Internet Explorer with TLS 1.0 enabled all failed to connect. Fixed by using **Pale Moon** browser, which was compatible with the legacy SSL version.

Configured the SSID (`SouthGroup`), connected a laptop to the Wi-Fi, confirmed it received a DHCP address from Server1, and verified connectivity to `south.com`.

---

## Phase 8 — VLANs & Router-on-a-Stick

Partner configured VLAN 10, 20, and 30 on the switch and disabled unused VLANs. After VLANs were applied, routing stopped working across the network.

**Steps taken to restore and complete routing:**

1. Identified the switch port connected to the router (`Gi1/0/4`) and set it to trunk mode carrying VLAN tags 10, 20, and 30.
2. Created router sub-interfaces on `Gi0/0` (`.10`, `.20`, `.30`) as virtual gateways for each VLAN.
3. Configured DHCP Relay on each sub-interface:
   ```
   ip helper-address 192.168.13.130
   ```
4. Replaced the original single DHCP scope with three new scopes on Server1, one per VLAN, each with subnet mask `255.255.255.192`:

   | Scope | Range | Gateway |
   |---|---|---|
   | VLAN 10 | 192.168.13.10 – .62 | 192.168.13.1 |
   | VLAN 20 | 192.168.13.75 – .126 | 192.168.13.65 |
   | VLAN 30 | 192.168.13.140 – .190 | 192.168.13.129 |

5. Fixed workstation VLAN port assignments on the switch:
   ```
   switchport access vlan [XX]
   ```
6. Ran `ipconfig /renew` on both workstations to pull the correct IPs from the new scopes.

---

## Phase 9 — Inter-Group Routing & ACL

Configured the router's outside interface (`192.168.6.1`) and added a static route to Group 2's router (`192.168.6.7`).

> **Issue:** Started receiving duplicate IP errors for `192.168.13.129` on the router and switch. After troubleshooting, found that an old unused VLAN interface on the switch still had that IP assigned from an earlier configuration. Removed the IP from the unused VLAN and shut it down — errors resolved.

Added DNS records to resolve Group 2's mail server (`mail.north.com`). Assisted Group 2 with their email configuration. Verified ACL blocking port 80 from outside was active.

---

## Phase 10 — Final Cleanup

Factory reset all equipment and stored everything in an organized manner as required by the instructor.
