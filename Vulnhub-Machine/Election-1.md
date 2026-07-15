# Election: 1

- **Machine:** Election: 1
- **Download:** https://www.vulnhub.com/entry/election-1,503/

![](703-1.png)

---

## Setup

- Extract the downloaded archive.
- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.31.0/24
```

![](703-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -p- 192.168.31.60
```

![](703-3.png)

---

## Service & Version Detection

```bash
nmap -sC -sV -A 192.168.31.60
```

![](703-4.png)

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.31.60
```

![](703-5.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.31.60/
- http://192.168.31.60/phpinfo.php
- http://192.168.31.60/phpmyadmin/
- http://192.168.31.60/robots.txt

![](703-6.png)

---

## Explore the Election Application

Visit:

- http://192.168.31.60/election/

![](703-7.png)

---

## Directory Enumeration

Perform directory brute-forcing against the Election application.

```bash
gobuster dir -u http://192.168.31.60/election/ -w /usr/share/wordlists/dirb/common.txt
```

![](703-8.png)

---

## Interesting Directories

Visit the discovered directories:

- http://192.168.31.60/election/data/
- http://192.168.31.60/election/admin/

![](703-9.png)

---

## Enumerate the Admin Panel

Continue directory enumeration inside the `/admin` directory.

```bash
gobuster dir -u http://192.168.31.60/election/admin/ -w /usr/share/wordlists/dirb/common.txt
```

![](703-10.png)

---

## Discover Log Files

Visit the discovered log directory.

- http://192.168.31.60/election/admin/logs/

![](703-11.png)

Open the system log file.

- http://192.168.31.60/election/admin/logs/system.log

![](703-12.png)

The log file contains valid user credentials.

---

# SSH Access

Use the credentials recovered from the log file to authenticate via SSH.

```bash
ssh love@192.168.31.60
```

### Credentials

```text
Username : love
Password : P@$$w0rd@123
```

![](703-13.png)

Successful authentication provides an interactive shell.

---

# Key Findings

- Open ports **22 (SSH)** and **80 (HTTP)** were identified during enumeration.
- Sensitive log files were publicly accessible through the web application.
- Valid SSH credentials were disclosed inside the log file.
- SSH access was obtained without attacking the login panel.

---

# Impact

- Sensitive information disclosure through exposed log files.
- Disclosure of valid user credentials.
- Unauthorized SSH access to the target system.
- Initial foothold obtained through insecure log management.

---

# Key Learning

- Always enumerate hidden directories within administrative panels.
- Log files frequently contain sensitive operational information.
- Credentials discovered in web applications should be tested against other exposed services.
- Administrative logs should never be publicly accessible.
- Proper access controls are essential to prevent credential leakage.

---

# Summary

The assessment began with web enumeration of the Election application, revealing multiple administrative directories. Further enumeration exposed a publicly accessible system log containing valid SSH credentials. Rather than attacking the web login interface, the recovered credentials were successfully used to authenticate directly over SSH, demonstrating how sensitive log files can lead to complete initial system access.