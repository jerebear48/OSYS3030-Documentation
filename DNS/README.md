# DNS Configuration — BIND9 on Ubuntu Server

## Overview
This directory contains configuration details for the **BIND9 DNS service** running on an Ubuntu Server VM.  
The server hosts an internal DNS zone for **`innovate.lab`**, providing A, AAAA, CNAME, and wildcard records to support internal name resolution and testing.

---

## Service Summary
| Setting | Value |
|----------|--------|
| **Service** | BIND9 (named) |
| **Domain** | `innovate.lab` |
| **Server IP** | `172.16.144.100` |
| **Zone File** | `/etc/bind/db.innovate.lab` |
| **Config File** | `/etc/bind/named.conf.local` |
| **Firewall Ports** | 53/UDP, 53/TCP |

---

## Configuration Process

### 1️ Install Required Packages
Install BIND9 and DNS utilities:
```bash
sudo apt update
sudo apt install bind9 dnsutils -y
```
Ensure the service is active:
```
sudo systemctl status bind9
```
## 2 Define the DNS Zone

Open the local BIND configuration file:
```
sudo nano /etc/bind/named.conf.local
```

Add the following zone block:
```
zone "innovate.lab" {
    type master;
    file "/etc/bind/db.innovate.lab";
};
```
Save and exit.

## 3️ Create and Edit the Zone File

Copy a default template and modify it for the innovate.lab domain:
```
sudo cp /etc/bind/db.local /etc/bind/db.innovate.lab
sudo nano /etc/bind/db.innovate.lab
```

Example contents:
```
;
; BIND data file for innovate.lab
;
$TTL    604800
@       IN      SOA     ns1.innovate.lab. admin.innovate.lab. (
                                  4         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.innovate.lab.

; A and AAAA records
ns1     IN      A       172.16.144.100
ns1     IN      AAAA    ::1

; Host records
www     IN      A       172.16.144.100
api     IN      A       172.16.144.100
resolver IN     CNAME   ns1.innovate.lab.

; Wildcard record
*       IN      A       10.10.10.50

; SPF record for outbound mail
@       IN      TXT     "v=spf1 include:_spf.google.com ~all"
```

## Validate and Reload Configuration

Run syntax checks to confirm there are no errors:
```
sudo named-checkconf
sudo named-checkzone innovate.lab /etc/bind/db.innovate.lab
```

If successful, reload the BIND9 service:
```
sudo systemctl reload bind9
sudo systemctl status bind9
```

## Verification
### Query Local DNS Records

Use dig to confirm proper resolution:
```
dig @localhost ns1.innovate.lab
dig @localhost resolver.innovate.lab
dig @localhost api.innovate.lab
dig @localhost anything.innovate.lab
```
Expected results:

* ns1, www, and api → resolve to 172.16.144.100
* resolver → CNAME → ns1.innovate.lab
*anything.innovate.lab → resolves to wildcard IP 10.10.10.50

Files Included
```
dns/
├── README.md
├── named.conf.local
└── db.innovate.lab
```

## Summary
This configuration enables the server to act as an internal authoritative DNS host for innovate.lab.
It supports:
* A / AAAA records for hosts
* CNAMEs for internal aliasing
* Wildcard resolution for dynamic subdomains
* SPF record for mail service compatibility

Together, these components provide a flexible and secure internal DNS service suitable for isolated lab and development environments.
