# Build Process & Troubleshooting

[← Back to README](README.md)

---

## Phase 1 — Planning & Physical Setup

Designed the initial network diagram including device roles, IP addressing, VLANs, and subnetting. Received physical hardware, installed it in server racks, ran cables, reset workstations, and installed Windows 11 and Windows Server 2022.

---

## Phase 2 — Switch & Basic Connectivity

Connected via console cable to configure the switch. The switch hadn't been properly reset from a previous year — had to swap it out for a new one. Configured static IPs and verified basic ping connectivity across devices.

---

## Phase 3 — Hypervisor Issues & Pivot to Bare-Metal

Originally planned to run AD/DNS/DHCP and the email server as two VMs on Server1. Hit repeated BSODs trying different ISOs and configurations. Switched to VMware Workstation Pro and got VMs running, but the host server was randomly shutting down.

After consulting the instructor, learned that a Type 2 hypervisor was causing instability on the hardware. Pivoted to a bare-metal installation of Windows Server 2022 directly on Server2 instead.

---

## Phase 4 — Domain Controller & DNS

Promoted Server1 to Domain Controller and created the `south.com` forest. Configured DNS with an A record and MX record for `mail.south.com → 192.168.13.131`.

> **Issue:** VM bridged adapters were set to automatic NIC selection — had to manually pin them to the host server's NIC in VMware settings before the pivot to bare-metal resolved the problem entirely.

---

## Phase 5 — DHCP & Workstations

Configured DHCP scope on Server1 (`192.168.13.50 – .100`). Joined both workstations to `south.com`.

> **Issue:** Workstations prompted for a password reset on first login even though no password had been set. Resolved through troubleshooting the default domain policy. Verified DHCP leases and domain login after fix.

---

## Phase 6 — Email Server (hMailServer)

Installed hMailServer on Server2 to handle SMTP, IMAP, and POP3.

> **Issue:** The installer required .NET Framework 2.0, which the server couldn't install normally. Fixed by mounting a Windows Server ISO and running:
> ```powershell
> Install-WindowsFeature Net-Framework-Core -source D:\sources\sxs
> ```

> **Issue:** Server2 was a clone of Server1 and had a duplicate SID, which prevented it from joining the `south.com` domain. Fixed by resetting the SID using Sysprep before joining the domain.

Configured hMailServer with the `south.com` domain, created two user accounts (`leandro@south.com`, `steven@south.com`), and enabled SMTP (25), IMAP (143), and POP3 (110). Added inbound firewall rules for those ports.

Installed Mozilla Thunderbird on both workstations. Configured incoming server as `mail.south.com` on IMAP port 143 and outgoing as `mail.south.com` on SMTP port 25.

Successfully sent an internal email from `leandro@south.com` to `steven@south.com`.

---

## Phase 7 — WAP Configuration

The WAP's CLI was unsupported, and the web GUI required an SSL version too old for modern browsers.

> **Issue:** Chrome, Edge (with security disabled), and Internet Explorer with TLS 1.0 enabled all failed to load the WAP GUI. Fixed by using **Pale Moon** browser, which was compatible with the legacy SSL.

Configured the SSID (`SouthGroup`), connected a laptop to the Wi-Fi, confirmed it received a DHCP address from Server1, and verified connectivity to `south.com`.

---

## Phase 8 — VLANs & Router-on-a-Stick

Partner configured VLAN 10, 20, and 30 on the switch and disabled unused VLANs. After VLAN implementation was applied, routing stopped working.

**Steps taken to restore and complete routing:**

1. Identified the switch port connected to the router (`Gi1/0/4`) and set it to trunk mode with VLAN tags 10, 20, and 30.
2. Created router sub-interfaces on `Gi0/0` (`.10`, `.20`, `.30`) as virtual gateways for each VLAN.
3. Configured DHCP Relay on each sub-interface:
   ```
   ip helper-address 192.168.13.130
   ```
4. Created three new DHCP scopes on Server1 with the correct `/26` subnet masks and gateway options matching the new VLAN plan.
5. Fixed workstation VLAN port assignments on the switch with:
   ```
   switchport access vlan [XX]
   ```
6. Ran `ipconfig /renew` on both workstations to pull the correct IPs.

---

## Phase 9 — Inter-Group Routing & ACL

Configured the router's outside interface (`192.168.6.1`) and added a static route to Group 2's router (`192.168.6.7`).

> **Issue:** Started receiving duplicate IP errors for `192.168.13.129` on both the router and switch. After troubleshooting, found that an old unused VLAN interface on the switch still had that IP assigned from before the router-on-a-stick reconfiguration. Removed the IP from the unused VLAN and shut it down — errors resolved.

Added DNS records to resolve Group 2's mail server (`mail.north.com`). Assisted Group 2 with their email configuration. Verified ACL blocking port 80 from outside was active.

---

## Phase 10 — Final Cleanup

Factory reset all equipment and stored devices in an organized manner as required.
