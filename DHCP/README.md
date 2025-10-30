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
