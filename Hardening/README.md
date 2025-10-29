# Post-Installation and Server Hardening

##  Overview
This lab focuses on the critical steps performed **immediately after installing Ubuntu Server 22.04 LTS** on a Proxmox virtual machine.  
The goal is to secure remote access, lock down user permissions, keep software updated, and implement a basic firewall.  
These actions form the foundation of good configuration management and server hygiene.

---

## Part 1 — SSH Key-Based Authentication

### Generate an SSH Key Pair
On your **local workstation** or host machine:
```bash
  ssh-keygen
```
Record the full path to your .pub file — it’s typically located in ~/.ssh/id_rsa.pub.

Copy Your Public Key to the Server
* On Linux / WSL:
```bash
  ssh-copy-id -i ~/.ssh/id_rsa.pub user@server_ip
```
* On Windows PowerShell:
```bash
type C:\Users\<user>\.ssh\id_rsa.pub | ssh user@server_ip "cat >> .ssh/authorized_keys"
```
Test Password-less SSH Access
```bash
  ssh user@server_ip
```
You should log in without being prompted for a password.

## Part 2 — Create a Local SSH Profile

For faster logins, add your connection details to your SSH config file:
```
  nano ~/.ssh/config
```
Example:
```
  Host ubuntu-server
      HostName 172.16.144.100
      User jeremy
      Port 2222
```
Now you can simply connect with:
```
  ssh ubuntu-server
```

## Part 3 — Know Your Host
# Identify Running Services
```
  service --status-all | grep "+"
```

Review System Logs
Below are common log files to review and monitor regularly:
* /var/log/messages	General system activity
* /var/log/auth.log	Login attempts & SSH authentication
* /var/log/syslog	System event messages
* /var/log/apt/	Package management activity

Monitor logs live:
```
  sudo tail -f /var/log/syslog
```
Check for Rootkits
```
  sudo apt install rkhunter -y
  sudo rkhunter --check
```
Audit System Access
* View accounts with root privileges
```
  awk -F: '($3=="0"){print}' /etc/passwd
```
* Find users with blank passwords
```
  sudo awk -F: '($2==""){print $1}' /etc/shadow
```
* Lock a user account
```
  sudo passwd -l username
```

## Part 4 — Keep the System Up-to-Date

Apply Updates and Patches
```
  sudo apt update
  sudo apt list --upgradable
  sudo apt upgrade -y
  sudo apt dist-upgrade -y
```
Enable Unattended Upgrades
```
  sudo apt install unattended-upgrades -y
  sudo dpkg-reconfigure -plow unattended-upgrades
```
Keeping software current mitigates the majority of known vulnerabilities.

## Part 5 — SSH Service Hardening
Edit the SSH Configuration
```
  sudo nano /etc/ssh/sshd_config
```
Ensure the following settings are present:
```yaml
  Port 2222
  AddressFamily inet
  PermitRootLogin no
  PasswordAuthentication no
```
Restart SSH:
```bash
  sudo systemctl restart ssh
```
 - Open a new terminal to test before closing your current session:
  ```
    ssh user@server_ip -p 2222
  ```
## Part 6 — Firewall Configuration (UFW)
Install and Enable UFW
```
  sudo apt install ufw -y
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2222/tcp
  sudo ufw enable
  sudo ufw status verbose
```

Check Open Ports
```
  sudo ss -tupln
```
Review that only expected ports (e.g. 2222) are open.
-  Review firewall rules regularly as new services are added.

📄 Files Included
```
  hardening/
  ├── README.md        # This file
  ├── sshd_config      # Modified SSH configuration
  └── user.rules       # Example sudo/access restrictions
```

Summary

By following these steps, the server now:
* Uses key-based SSH authentication on port 2222
* Denies root and password-based logins
* Automatically applies security updates
* Restricts inbound access with UFW
* Implements standard system audit practices

These steps establish a secure, maintainable baseline for all future configuration work.

