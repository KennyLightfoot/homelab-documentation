# Project 3: VLAN Network Segmentation with pfSense

![Status](https://img.shields.io/badge/Status-Complete-success)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)

## 📋 Project Overview

Implemented enterprise-grade network segmentation using pfSense as a virtual router to isolate homelab infrastructure from production network traffic. This project demonstrates network architecture design, firewall rule configuration, inter-VLAN routing, and advanced network troubleshooting skills.

**Business Context:** Working from home running a mobile notary business required complete isolation between experimental lab traffic and production business network to ensure zero disruption to client services.

---

## 🎯 Objectives

- [x] Deploy pfSense as a virtual router in VMware Workstation Pro
- [x] Create isolated network segments (VLANs) for Production and Homelab traffic
- [x] Configure inter-VLAN routing with granular firewall rules
- [x] Migrate existing services (Pi-hole, Uptime Kuma, Portainer) to isolated VLAN
- [x] Enable controlled access from Production to specific Homelab services
- [x] Maintain security isolation (Homelab cannot reach Production)

---

## 🏗️ Architecture

### Network Topology

```
                        ┌─────────────────────────────────────────────────────────┐
                        │                    INTERNET                              │
                        └─────────────────────────┬───────────────────────────────┘
                                                  │
                        ┌─────────────────────────▼───────────────────────────────┐
                        │              AT&T BGW320 Gateway                         │
                        │              192.168.1.254/24                            │
                        └─────────────────────────┬───────────────────────────────┘
                                                  │
┌─────────────────────────────────────────────────┼─────────────────────────────────────────────────┐
│                                    VMware Workstation Pro                                         │
│  ┌──────────────────────────────────────────────┼──────────────────────────────────────────────┐  │
│  │                              pfSense Virtual Router                                          │  │
│  │                                                                                              │  │
│  │    ┌─────────────┐         ┌─────────────┐         ┌─────────────┐                          │  │
│  │    │  WAN (em0)  │         │  LAN (em1)  │         │HOMELAB(em2) │                          │  │
│  │    │   Bridged   │         │   VMnet2    │         │   VMnet3    │                          │  │
│  │    │192.168.1.233│         │192.168.10.1 │         │192.168.20.1 │                          │  │
│  │    └──────┬──────┘         └──────┬──────┘         └──────┬──────┘                          │  │
│  └───────────┼───────────────────────┼───────────────────────┼──────────────────────────────────┘  │
│              │                       │                       │                                     │
│              │               ┌───────┴───────┐       ┌───────┴───────┐                             │
│              │               │    VMnet2     │       │    VMnet3     │                             │
│              │               │ 192.168.10.0  │       │ 192.168.20.0  │                             │
│              │               │   VLAN 10     │       │   VLAN 20     │                             │
│              │               │  Production   │       │   Homelab     │                             │
│              │               └───────┬───────┘       └───────┬───────┘                             │
│              │                       │                       │                                     │
│              │               ┌───────┴───────┐       ┌───────┴───────────────┐                     │
│              │               │ Windows PC    │       │                       │                     │
│              │               │192.168.10.50  │       │  ┌─────────────────┐  │                     │
│              │               │ (VMnet2 NIC)  │       │  │  Pi-hole Server │  │                     │
│              │               └───────────────┘       │  │  192.168.20.10  │  │                     │
│              │                                       │  │  DNS + Adblock  │  │                     │
│              │                                       │  └─────────────────┘  │                     │
│              │                                       │                       │                     │
│              │                                       │  ┌─────────────────┐  │                     │
│              │                                       │  │Ubuntu-Server-Lab│  │                     │
│              │                                       │  │  192.168.20.11  │  │                     │
│              │                                       │  │ Uptime Kuma     │  │                     │
│              │                                       │  │ Portainer       │  │                     │
│              │                                       │  └─────────────────┘  │                     │
│              │                                       └───────────────────────┘                     │
└──────────────┼─────────────────────────────────────────────────────────────────────────────────────┘
               │
    ┌──────────▼──────────┐
    │  Physical Network   │
    │   192.168.1.0/24    │
    │  (Business Traffic) │
    └─────────────────────┘
```

### Network Segments

| Segment | Subnet | Gateway | Purpose |
|---------|--------|---------|---------|
| WAN | 192.168.1.0/24 | 192.168.1.254 | Internet access via AT&T gateway |
| VLAN 10 (Production) | 192.168.10.0/24 | 192.168.10.1 | Windows workstation access |
| VLAN 20 (Homelab) | 192.168.20.0/24 | 192.168.20.1 | Lab services and infrastructure |

---

## 🛠️ Technical Implementation

### pfSense Virtual Router Configuration

**VM Specifications:**
- **Hypervisor:** VMware Workstation Pro
- **OS:** pfSense 2.8.1-RELEASE
- **RAM:** 2GB
- **CPU:** 2 cores
- **Storage:** 20GB

**Network Interfaces:**

| Interface | VMware Network | IP Address | DHCP Range |
|-----------|---------------|------------|------------|
| WAN (em0) | Bridged | DHCP (192.168.1.233) | N/A |
| LAN (em1) | VMnet2 (Host-only) | 192.168.10.1/24 | 192.168.10.100-200 |
| HOMELAB (em2) | VMnet3 (Host-only) | 192.168.20.1/24 | 192.168.20.100-200 |

### VMware Virtual Networks

```
VMnet2 (LAN/Production):
├── Subnet: 192.168.10.0/24
├── Type: Host-only
├── DHCP: Disabled (pfSense handles DHCP)
└── Host adapter: Connected (allows Windows access)

VMnet3 (HOMELAB):
├── Subnet: 192.168.20.0/24
├── Type: Host-only
├── DHCP: Disabled (pfSense handles DHCP)
└── Host adapter: Connected
```

### Firewall Rules

**HOMELAB Interface (Isolation):**

| Order | Action | Protocol | Source | Destination | Purpose |
|-------|--------|----------|--------|-------------|---------|
| 1 | Block | Any | HOMELAB subnets | LAN subnets | Prevent lab → production |
| 2 | Allow | Any | HOMELAB net | Any | Internet access for lab |

**LAN Interface (Controlled Access):**

| Order | Action | Protocol | Source | Destination | Purpose |
|-------|--------|----------|--------|-------------|---------|
| 1 | Allow | TCP | LAN subnets | 192.168.20.11:9443 | Portainer HTTPS |
| 2 | Allow | TCP | LAN subnets | 192.168.20.11:3001 | Uptime Kuma |
| 3 | Allow | TCP | LAN subnets | 192.168.20.10:8080 | Pi-hole Web UI |
| 4 | Allow | TCP/UDP | LAN subnets | 192.168.20.10:53 | Pi-hole DNS |
| 5 | Allow | Any | LAN net | Any | Default LAN access |

### Windows Routing Configuration

Static route added to Windows host for HOMELAB access:

```powershell
# Route traffic destined for HOMELAB through pfSense LAN interface
New-NetRoute -DestinationPrefix "192.168.20.0/24" -InterfaceIndex 48 -NextHop "192.168.10.1"
```

**Verification:**
```powershell
PS> Find-NetRoute -RemoteIPAddress 192.168.20.11

IPAddress         : 192.168.10.50
InterfaceAlias    : VMware Network Adapter VMnet2
NextHop           : 192.168.10.1
DestinationPrefix : 192.168.20.0/24
```

---

## 🔧 Troubleshooting Journey

This project involved extensive troubleshooting that demonstrates real-world network diagnostic skills.

### Issue 1: Windows Route Not Persisting

**Symptom:** Static route for 192.168.20.0/24 disappeared after being added.

**Diagnosis:**
```powershell
# Route existed in routing table but wasn't being used
Get-NetRoute -DestinationPrefix "192.168.20.0/24"
# Error: No objects found

# Windows was routing through wrong interface
Find-NetRoute -RemoteIPAddress 192.168.20.11
# Showed Ethernet 2 (wrong) instead of VMnet2
```

**Resolution:** Re-added route with correct interface index using PowerShell (admin):
```powershell
New-NetRoute -DestinationPrefix "192.168.20.0/24" -InterfaceIndex 48 -NextHop "192.168.10.1"
```

### Issue 2: Packets Not Reaching pfSense

**Symptom:** Ping to 192.168.20.11 timed out despite correct Windows routing.

**Diagnosis using Packet Capture:**

```
pfSense: Diagnostics → Packet Capture
Interface: LAN (em1)
Filter: Host 192.168.10.50
```

**Finding:** No ICMP packets from Windows appeared on pfSense LAN interface.

**Advanced Diagnosis using Windows pktmon:**
```powershell
pktmon filter add -i 192.168.20.11
pktmon start --etw -m real-time
```

**Key Finding:**
```
Direction Tx: 192.168.10.50 > 192.168.20.11: ICMP echo request
Direction Rx: ARP, Request who-has 192.168.20.11 tell 192.168.20.1
```

**Interpretation:** 
- Windows WAS sending packets correctly ✅
- pfSense WAS receiving and forwarding ✅
- pfSense was ARPing for 192.168.20.11 but getting no response ❌

### Issue 3: Netplan Interface Name Mismatch (Root Cause)

**Symptom:** pfSense couldn't reach Ubuntu-Server-Lab at 192.168.20.11.

**Diagnosis:**
```bash
# On Ubuntu-Server-Lab
ip addr show ens33
# Showed: inet 192.168.20.101/24 (DHCP address, not static!)

cat /etc/netplan/50-cloud-init.yaml
# Showed interface configured as "ens303" instead of "ens33"
```

**Root Cause:** Netplan configuration file had wrong interface name (`ens303` instead of `ens33`), causing static IP configuration to fail and DHCP to assign `.101` instead of `.11`.

**Resolution:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
# Changed: ens303 → ens33

sudo netplan apply

ip addr show ens33
# Now shows: inet 192.168.20.11/24 ✅
```

---

## ✅ Results

### Connectivity Verification

**Windows → Homelab Services:**
```powershell
PS> ping 192.168.20.11 -n 4
Reply from 192.168.20.11: bytes=32 time=1ms TTL=63
Reply from 192.168.20.11: bytes=32 time=7ms TTL=63
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss) ✅

PS> Test-NetConnection -ComputerName 192.168.20.11 -Port 3001
TcpTestSucceeded : True ✅
```

**Security Isolation (Homelab → Production):**
```bash
# From Ubuntu-Server-Lab
ping 192.168.10.50 -c 2
# 100% packet loss ✅ (blocked by firewall as designed)
```

### Accessible Services

| Service | URL | Status |
|---------|-----|--------|
| Uptime Kuma | http://192.168.20.11:3001 | ✅ Accessible |
| Portainer | https://192.168.20.11:9443 | ✅ Accessible |
| Pi-hole | http://192.168.20.10:8080 | ✅ Accessible |

---

## 📚 Key Learnings

### Technical Skills Gained

1. **pfSense Administration**
   - Virtual router deployment and multi-interface configuration
   - Firewall rule creation with proper ordering
   - DHCP server configuration per interface
   - Packet capture for network diagnostics

2. **Network Segmentation Concepts**
   - VLAN design for security isolation
   - Inter-VLAN routing principles
   - One-way access control (allow Production→Lab, block Lab→Production)

3. **Advanced Troubleshooting**
   - Layer-by-layer packet tracing (Windows → pfSense → VM)
   - Using `pktmon` for Windows packet monitoring
   - pfSense packet capture analysis
   - ARP troubleshooting

4. **Windows Networking**
   - Static route management with PowerShell
   - Multi-adapter routing behavior
   - `Find-NetRoute` for route verification

5. **Linux Networking**
   - Netplan configuration for static IPs
   - Interface naming and configuration
   - `tcpdump` for packet analysis

### Enterprise Relevance

This architecture mirrors enterprise network designs where:
- Development/Test environments are isolated from Production
- Specific services are exposed through controlled firewall rules
- Network segmentation provides defense-in-depth security
- Troubleshooting requires systematic packet-level analysis

---

## 🎤 Interview Talking Points

**Q: "Tell me about a complex networking problem you solved."**

> "In my homelab, I implemented VLAN segmentation using pfSense to isolate lab infrastructure from my production business network. After configuring everything, connectivity wasn't working despite correct firewall rules. I used a systematic approach - packet captures on pfSense showed traffic arriving but not being forwarded. Using Windows' pktmon tool, I traced packets and discovered pfSense was sending ARP requests for a VM that wasn't responding. The root cause was a netplan configuration file with a typo in the interface name, causing the VM to get a DHCP address instead of the expected static IP. This taught me the importance of layer-by-layer troubleshooting and verifying every component in the network path."

**Q: "How do you approach network troubleshooting?"**

> "I follow a systematic, layer-by-layer approach. First, I verify physical/virtual connectivity, then check IP addressing and routing tables. I use packet captures at multiple points to see exactly where traffic stops flowing. In this project, I captured packets on the Windows host, on pfSense's LAN interface, and on pfSense's HOMELAB interface. By comparing what I saw at each point, I could pinpoint exactly where the problem occurred - traffic was reaching pfSense but the destination VM wasn't responding to ARP requests because it had the wrong IP address."

**Q: "Why is network segmentation important?"**

> "Network segmentation provides defense-in-depth security by limiting the blast radius of any security incident. In my homelab, I needed to ensure experimental configurations couldn't disrupt my production business network. By placing lab services on a separate VLAN with firewall rules blocking lab-to-production traffic, I created a secure boundary. This mirrors enterprise environments where you'd separate development from production, guest networks from corporate, or IoT devices from sensitive systems."

---

## 🚀 Next Steps

- [ ] Implement Pi-hole as DNS server for Windows (using 192.168.20.10)
- [ ] Add monitoring for inter-VLAN traffic in Uptime Kuma
- [ ] Document network topology in draw.io for portfolio
- [ ] Configure pfSense logging to future Wazuh SIEM (Project 4)
- [ ] Clean up netplan configuration warnings on Ubuntu-Server-Lab

---

## 🔗 Resources

- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [VMware Virtual Networks](https://docs.vmware.com/en/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-D9B0A52D-38A2-45D7-A9EB-987ACE77F93C.html)
- [Ubuntu Netplan Configuration](https://netplan.io/reference)
- [Windows Routing Commands](https://docs.microsoft.com/en-us/powershell/module/nettcpip/new-netroute)

---

## 📊 Project Stats

| Metric | Value |
|--------|-------|
| **Completion Date** | January 25, 2026 |
| **Time Investment** | ~4 hours |
| **Difficulty Rating** | Advanced |
| **VMs Deployed** | 3 (pfSense, Pi-hole-Server, Ubuntu-Server-Lab) |
| **Firewall Rules Created** | 7 |
| **Troubleshooting Tools Used** | pktmon, tcpdump, pfSense packet capture, PowerShell |

---

*This project is part of my [Homelab Documentation Portfolio](https://github.com/KennyLightfoot/homelab-documentation) demonstrating hands-on cloud engineering and DevOps skills.*
