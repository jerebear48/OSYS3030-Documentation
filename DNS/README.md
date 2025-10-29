#  DNS Configuration — BIND9 on Ubuntu Server

##  Overview
This directory documents the DNS configuration for the Ubuntu Server appliance.  
The server uses **BIND9** to provide name resolution for the internal domain **`innovate.lab`** and includes a wildcard record for dynamic subdomain resolution.

---

##  Service Description
**Service:** Domain Name System (DNS)  
**Software:** BIND9  
**Domain:** `innovate.lab`  
**Server IP:** `172.16.144.#`  
**Primary Zone File:** `/etc/bind/db.innovate.lab`

The server is authoritative for `innovate.lab` and handles internal lookups, CNAME records, and wildcard resolution for development subdomains.

---

##  Configuration Details

### Zone Declaration
Defined in `/etc/bind/named.conf.local`:
```bash
zone "innovate.lab" {
    type master;
    file "/etc/bind/db.innovate.lab";
};
```

Zone File Example
Located at /etc/bind/db.innovate.lab:
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

; Wildcard for undefined subdomains
*       IN      A       10.10.10.50

; SPF record for mail integration
@       IN      TXT     "v=spf1 include:_spf.google.com ~all"
```

## Verification Commands
Check Configuration Syntax
```
sudo named-checkconf
sudo named-checkzone innovate.lab /etc/bind/db.innovate.lab
```
Reload BIND9
```
sudo systemctl reload bind9
sudo systemctl status bind9
```
Test Record Resolution
```
dig @localhost ns1.innovate.lab
dig @localhost resolver.innovate.lab
dig @localhost api.innovate.lab
dig @localhost anything.innovate.lab
```

Expected Results:
* Defined records (e.g., ns1, api, www) resolve to 172.16.144.100
* Undefined subdomains (e.g., anything.innovate.lab) resolve to 10.10.10.50

Notes
* The wildcard record simplifies internal testing by resolving any undefined hostname within the innovate.lab zone.
* SPF records are included for mail service compatibility with Google Apps.
* Only port 53 (UDP/TCP) should be open through the firewall:
```
sudo ufw allow 53
```

Files Included
```
dns/
├── README.md
├── named.conf.local
└── db.innovate.lab
```
