# 🖥️ Server Configuration Appliance

## 📘 Description
This server is an **Ubuntu Server 22.04 LTS** appliance used to provide and manage core network services in a lab environment.  
It functions as a central system for **DNS**, **DHCP**, and **SSH** access, and includes system hardening and firewall rules for security.

---

## 🌐 Network Services Overview
| Service | Description |
|----------|-------------|
| **SSH** | Provides secure remote access using key-based authentication and a custom port. |
| **DNS (BIND9)** | Resolves local domain names within the `innovate.lab` zone. |
| **DHCP (Kea)** | Automatically assigns IP addresses to clients on the internal network. |
| **Firewall (UFW)** | Restricts and filters network traffic to only required ports. |
| **NTP (Chrony)** | Keeps the system clock synchronized with network time servers. |

---

## 🧭 Network Topology
This appliance supports two network interfaces — one for WAN access and one for LAN connectivity.

[ Clients ] ←→ [ ens19: LAN (10.x.x.0/24) ] ←→ [ Ubuntu Server ] ←→ [ ens18: WAN (172.16.144.0/24) ] ←→ [ Internet ]


- **ens18** – External/WAN interface  
- **ens19** – Internal/LAN interface  

---

## ⚙️ Installation Notes
1. Install **Ubuntu Server 22.04 LTS** (minimal installation recommended).  
2. Enable **OpenSSH Server** during installation.  
3. After the first boot, update the system:
 ```bash
  sudo apt update && sudo apt upgrade -y
```

Install Required Packages
```bash
  sudo apt install bind9 dnsutils isc-kea-dhcp4-server ufw chrony -y
```

Configure Networking
Use Netplan to assign static IP addresses for both interfaces:
```bash
  sudo nano /etc/netplan/01-netcfg.yaml
  sudo netplan apply
```

Enable and Verify Services
```bash
  sudo systemctl enable bind9 --now
  sudo systemctl enable kea-dhcp4-server --now
  sudo ufw enable
  sudo ufw status verbose
```
