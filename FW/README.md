# Multi-Homed Network Configuration

## Objective
In this section, the server was configured with **two network interfaces** — one for external (WAN) access and one for internal (LAN) connectivity.  
This setup allows the server to function like a gateway or firewall, separating internal and external traffic for improved security and control.

---

## Virtual Hardware Setup
The server runs as a virtual machine on **Proxmox VE (PVE)** with **two network adapters** assigned:

| Interface | Role | Description |
|------------|------|-------------|
| `ens18` | WAN | Connected to the Proxmox bridge (external or management network). |
| `ens19` | LAN | Connected to an isolated internal network for client communication. |

If running on other hypervisors (VMware or VirtualBox), a second adapter must be created using **Host-only** or **Internal Network** mode before powering on the VM.

---

## Part 1 — Gather Network Information

Before editing configuration files, record current network details.

### Identify Interfaces
```bash
  sudo lshw -C network
```
Record IP, Gateway, and DNS
```bash
  ip a          # Show current IP addresses
  ip r          # Show routing table (gateway)
  resolvectl status   # Show DNS information
```

Write these down:

Primary IP Address: __________________
Default Gateway: __________________
DNS Server: __________________

## Part 2 — Configure Static Networking

Edit the Netplan configuration file to make the IP settings permanent.
```bash
  sudo nano /etc/netplan/01-netcfg.yaml
```

**Configure both interfaces.** Your file should be modified to look like the example below. **Replace the placeholder information** with your interface names and the IP details you gathered in Part 2.

- **WAN Interface:** This is your original adapter. Configure it with the static IP, gateway, and DNS information you recorded.
- **LAN Interface:** This is your new adapter. Configure it with the static IP `10.0.0.250/24`. It does **not** need a gateway or DNS servers.

```bash
  network:
    version: 2
    renderer: networkd
    ethernets:
      # This is the WAN (public-facing) interface
      ens18:  # <--- Is this the right interface?
        dhcp4: no
        addresses:
          - 192.168.10.123/24   # <-- Use the IP you recorded
        routes:
          - to: default
            via: 192.168.10.1     # <-- Use the Gateway you recorded
        nameservers:
          addresses: [192.168.10.1] # <-- Use the DNS you recorded
  
      # This is the LAN (private) interface
      ens19:    # <--- Is this the right interface?
        dhcp4: no
        addresses:
          - 10.0.0.250/24
```
Apply the configuration:

```bash
  sudo netplan apply
```
Netplan will display an error message if there are syntax issues.

## Part 3 — Verification

1. Check IP Addresses
```bash
  ip a
```

Both interfaces should show static addresses.

2. Test Gateway Connectivity
```bash
  ping <your_gateway_ip>
```
4. Test Internet Connectivity
```bash
  ping 8.8.8.8
```
5. Test DNS Resolution
```bash
  ping google.com
```

If all four tests pass:

* Both interfaces have correct static IPs
* Gateway responds
* Internet connectivity works
* DNS resolution succeeds

## Part 4 — Basic Firewall Setup

Enable and configure UFW for secure access:
```
  sudo apt install ufw -y
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2222/tcp     # SSH
  sudo ufw enable
```

Files Included
```
  fw/
  ├── README.md         # This file
  └── 01-netcfg.yaml    # Example Netplan configuration
```

The server is now configured as a multi-homed system capable of routing traffic between two networks while applying firewall restrictions to control external access.


