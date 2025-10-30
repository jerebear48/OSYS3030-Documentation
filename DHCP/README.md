# DHCP Service — Kea on Ubuntu Server

## Overview
This directory documents the **Kea DHCPv4** configuration used to provide dynamic IP addressing and DNS option delivery within an isolated Proxmox or virtualized network environment.

The server functions as a DHCP and DNS provider for clients on the `10.<ID>.0.0/24` network, with the DHCP service managed by **Kea** and leases stored in-memory via the `memfile` backend.

---

## Service Summary
| Setting | Value |
|----------|--------|
| **Service** | Kea DHCPv4 (`kea-dhcp4-server.service`) |
| **Interface** | `ens19` |
| **Server IP** | `10.<ID>.0.250/24` |
| **Subnet** | `10.<ID>.0.0/24` |
| **Pool Range** | `10.<ID>.0.10 – 10.<ID>.0.200` |
| **Default Gateway** | `10.<ID>.0.250` |
| **DNS Server** | `10.<ID>.0.250` |
| **Lease Database** | In-memory (`memfile`) |
| **Configuration File** | `/etc/kea/kea-dhcp4.conf` |

---

## Installation

Kea is available from the default Ubuntu repositories and can be installed alongside BIND9 or standalone.

* Do note that installing kea uninstalls UFW and can cause confilcts, make sure you back up your configs beforehand 

```bash
sudo apt update
sudo apt install kea-dhcp4-server -y
```
Enable and start the service:
```
sudo systemctl enable kea-dhcp4-server.service
sudo systemctl start kea-dhcp4-server.service
```
Verify that the service is active:
```
sudo systemctl status kea-dhcp4-server.service
```
If reinstalling or resetting configuration, the package can be purged and reinstalled cleanly:
```
sudo apt purge kea-dhcp4-server -y
sudo apt install kea-dhcp4-server -y
```
## Configuration Overview

Kea is configured to operate on a single private subnet, using interface ens19.
Clients receive IP addresses from a defined pool and are provided with gateway and DNS settings through DHCP Options 3 (Router) and 6 (DNS Server).

All configuration is stored in JSON format for clarity and automation compatibility.

Example Configuration
```
/etc/kea/kea-dhcp4.conf

{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": ["ens19"]
    },
    "lease-database": {
      "type": "memfile",
      "lfc-interval": 3600
    },
    "subnet4": [
      {
        "id": 1,
        "subnet": "10.<ID>.0.0/24",
        "pools": [
          { "pool": "10.<ID>.0.10 - 10.<ID>.0.200" }
        ],
        "option-data": [
          { "name": "routers", "data": "10.<ID>.0.250" },
          { "name": "domain-name-servers", "data": "10.<ID>.0.250" }
        ]
      }
    ],
    "renew-timer": 900,
    "rebind-timer": 1800,
    "valid-lifetime": 3600,
    "loggers": [
      {
        "name": "kea-dhcp4",
        "output_options": [
          { "output": "stdout", "pattern": "%-5p %m\n" }
        ],
        "severity": "INFO"
      }
    ]
  }
}
```
## Validation and Diagnostics

You can use the following commands to validate and diagnose your configuration. Make sure you populate commands with your adapters.

| Task |	Command |
|---------|----------------|
| Validate configuration syntax |	`cat /etc/kea/kea-dhcp4.conf |
| Restart and check service |	sudo systemctl restart kea-dhcp4-server && sudo systemctl status kea-dhcp4-server |
| View logs	sudo journal | ctl -u kea-dhcp4-server -f |
| Monitor DORA process | sudo tcpdump -i ens19 -n port 67 or port 68 |

## Network Integration

The Kea DHCP service operates on the same network segment as the internal DNS resolver (BIND9), ensuring that clients automatically receive the correct gateway and DNS IP (10.<ID>.0.250).
This configuration is typically deployed within an isolated Proxmox bridge (vmbr<ID>) or internal lab network.

## Design Notes
* Minimalist configuration — only one subnet defined.
* No reservations or relay configuration for simplicity.
* Uses the memfile backend for quick restart recovery and no external dependencies.
* Logging directed to stdout for compatibility with journalctl.

Repository Structure
```
dhcp/
├── README.md
└── kea-dhcp4.conf
```
This Kea DHCPv4 configuration provides a self-contained, isolated DHCP service for lab and development environments.
It delivers:

* Automatic IP address allocation (10.<ID>.0.10 – 10.<ID>.0.200)
* Gateway and DNS options for client auto-configuration
* Lightweight JSON-based configuration
* Seamless integration with internal DNS (innovate.lab)

This setup forms the foundation for isolated multi-host networking inside Proxmox or other virtualization platforms.
