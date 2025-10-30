# Server Configuration Appliance

## Description
This server is an **Ubuntu Server 22.04 LTS** appliance running as a virtual machine on a **Proxmox Virtual Environment (PVE)** host.  
It was created for lab use to practice system configuration management and network service setup.  
The server demonstrates essential administration tasks such as SSH hardening, firewall configuration, and static IP management across dual network interfaces.

---

## Network Services Overview
| Service | Description |
|----------|-------------|
| **SSH** | Provides secure remote access using key-based authentication and a custom port. |
| **DNS (BIND9)** | Resolves local domain names within the `innovate.lab` zone. |
| **DHCP (Kea)** | Automatically assigns IP addresses to clients on the internal network. |
| **Firewall (UFW)** | Restricts and filters network traffic to only required ports. |

---

## Network Topology
This appliance supports two network interfaces — one for WAN access and one for LAN connectivity.

[ Clients ] ←→ [ ens19: LAN (10.x.x.0/24) ] ←→ [ Ubuntu Server ] ←→ [ ens18: WAN (172.16.144.0/24) ] ←→ [ Internet ]


- **ens18** — External/WAN interface (bridged to Proxmox management or upstream network)  
- **ens19** — Internal/LAN interface (connected to isolated lab network or VLAN)

---

This setup allows the VM to simulate a real network appliance while remaining fully contained within a Proxmox virtualized lab.

---

## ⚙️ Initial Distro Installation

### Step 1 — Create the VM in Proxmox
- Use **Ubuntu Server 22.04 LTS ISO** as the installation image.  
- Assign **2 network adapters**:  
  - `vmbr0` → WAN / External  
  - `vmbr1` (or equivalent) → LAN / Internal   
- Allocate appropriate resources (2 vCPUs, 2–4 GB RAM, 16 GB storage).

### Step 2 — Install the Operating System
- Select **Minimal Installation** to reduce resource use.  
- Enable **OpenSSH Server** during setup.  

### Step 3 — Update the System
After first boot:
```bash
sudo apt update && sudo apt upgrade -y
```

Step 4 — Verify Connectivity

Ensure both interfaces function correctly:
```bash
 ip a
 ping 8.8.8.8
 ping google.com
```
