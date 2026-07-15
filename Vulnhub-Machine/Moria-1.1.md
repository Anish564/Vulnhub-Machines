# Moria: 1.1

- **Machine:** Moria: 1.1
- **Download:** https://www.vulnhub.com/entry/moria-1,187/

![](816-1.png)

---

# Setup

1. Extract the downloaded archive.
2. Import the `.ovf` file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](816-2.png)

---

## Full Nmap Scan

Run a complete scan to enumerate open ports, services, OS details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.192
```

![](816-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.192
```

```bash
nmap -sC -sV -A 192.168.2.192
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.192
```

![](816-4.png)

---

# Web Enumeration

Visit the target website.

- http://192.168.2.192

![](816-5.png)

A password hint is displayed.

```text
Password : mellon
```

---

## Explore Hidden Directories

Visit the following paths:

- http://192.168.2.192/w/
- http://192.168.2.192/w/h/i/s/p/e/r/the_abyss/

![](816-6.png)

---

## Directory Enumeration

Brute-force the hidden directory.

```bash
gobuster dir -u http://192.168.2.192/w/h/i/s/p/e/r/the_abyss/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](816-7.png)

Discovered endpoint:

- http://192.168.2.192/w/h/i/s/p/e/r/the_abyss/random.txt

![](816-8.png)

The file contains a list of usernames.

```text
Balin
Oin
Ori
Fundin
Nain
Maeglin
Telchar
Thrain
Dain
```

---

# FTP Enumeration

Attempt anonymous login.

```bash
ftp 192.168.2.192
```

![](816-9.png)

Anonymous login is not allowed.

Login using the discovered credentials.

```text
Username : Balrog
Password : Mellon69
```

```bash
ftp 192.168.2.192
```

![](816-10.png)

> **Note:** The password **Mellon69** works on version **1.1**, although it cannot be directly derived through enumeration.

---

## Browse FTP Files

Navigate to the web directory.

```bash
cd /var/www/html
```

List the available files.

```bash
ls
```

![](816-11.png)

A hidden directory is discovered.

Visit it in the browser.

- http://192.168.2.192/QlVraKW4fbIkXau9zkAPNGzviT3UKntl/

![](816-12.png)

Inspect the page source.

```text
view-source:http://192.168.2.192/QlVraKW4fbIkXau9zkAPNGzviT3UKntl/
```

![](816-13.png)

Several password hashes are exposed.

---

# Password Cracking

Save the hashes into a file.

```bash
nano hashes.txt
```

![](816-14.png)

Crack the hashes using John the Ripper.

```bash
john --format=dynamic_6 hashes.txt --wordlist=/opt/rockyou.txt
```

![](816-15.png)

Recovered credentials:

| Username | Password |
|----------|----------|
| Balin | flower |
| Nain | warrior |
| Ori | spanky |
| Dain | abcdef |
| Maeglin | fuckoff |
| Thrain | darkness |
| Telchar | magic |
| Fundin | hunter2 |

---

# SSH Access

Login using one of the recovered accounts.

```bash
ssh Ori@192.168.2.192
```

Password:

```text
spanky
```

![](816-16.png)

Successfully obtained SSH access.

---

# Key Learning

- Enumerate deeply nested directories.
- Inspect page source for hidden information.
- FTP access may expose sensitive web content.
- Always inspect files found through FTP.
- Crack password hashes using **John the Ripper**.
- Reuse recovered credentials to gain SSH access.

---

# Summary

The attack started with web enumeration, where a hidden directory exposed a list of valid usernames. FTP access using the available credentials revealed an additional hidden web directory containing password hashes. After cracking the hashes with **John the Ripper**, valid SSH credentials were recovered. These credentials provided successful SSH access to the target machine.