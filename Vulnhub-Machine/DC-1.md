# DC: 1

- **Machine:** DC: 1
- **Download:** https://www.vulnhub.com/entry/dc-1,292/

![](701-1.png)

---

## Setup

- Extract the downloaded archive.
- Import the `.ovf` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](701-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -p- 192.168.2.179
```

![](701-3.png)

---

## Service & Version Detection

```bash
nmap -sC -sV -A 192.168.2.179
```

![](701-4.png)

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.179
```

![](701-5.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.179/
- http://192.168.2.179/robots.txt

---

## Identify the Drupal Version

Use Nikto to enumerate the web application and identify the CMS version.

```bash
nikto -h http://192.168.2.179
```

![](701-6.png)

### Findings

- **CMS:** Drupal
- **Version:** Drupal 7

---

## Search for a Public Exploit

After identifying the Drupal version, search for a suitable public exploit.

Download the exploit from Exploit-DB:

- https://www.exploit-db.com/exploits/44449

![](701-7.png)

> **Note:** The exploit is written in Ruby.

---

# Initial Access (Lab)

> **Lab note:** The following step is intended only for an authorized practice machine such as this VulnHub VM.

Execute the Ruby exploit against the target.

```bash
ruby 44449.rb 192.168.2.179
```

![](701-8.png)

A reverse shell is successfully established.

---

# Impact

- Drupal 7 was identified as the target CMS.
- A publicly available exploit allowed remote code execution.
- Initial shell access to the target system was obtained.

---

# Key Learning

- Always identify the exact CMS and version during enumeration.
- Public exploit databases can be valuable during authorized penetration tests.
- Vulnerable and outdated CMS installations significantly increase the attack surface.
- Keeping web applications updated helps prevent known exploits.

---

# Summary

The assessment began with web enumeration, which identified the target as a Drupal 7 application. Version enumeration using Nikto confirmed the CMS version, allowing a matching public exploit to be located on Exploit-DB. Executing the Ruby-based exploit resulted in successful remote code execution and an initial shell on the target machine.