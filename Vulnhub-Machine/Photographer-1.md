# Photographer: 1

- **Machine:** Photographer: 1
- **Download:** https://www.vulnhub.com/entry/photographer-1,519/

![](images/752-1.png)

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

![](images/752-2.png)

---

## Port Scan

```bash
nmap -v -p- 192.168.2.122
```

![](images/752-3.png)

---

## Service Enumeration

```bash
nmap -sC -sV -A 192.168.2.122
```

![](images/752-4.png)

---

## HTTP Enumeration

Run the HTTP enumeration NSE script.

```bash
nmap -v -p 80 -sT -sV -A --script=http-enum.nse 192.168.2.122
```

![](images/752-5.png)

---

# SMB Enumeration

Enumerate available SMB shares.

```bash
smbclient -L //192.168.2.122/ -N
```

![](images/752-6.png)

Connect to the discovered share.

```bash
smbclient //192.168.2.122/sambashare -N
```

![](images/752-7.png)

Download the available files.

```bash
get mailsent.txt
```

```bash
get wordpress.bkp.zip
```

![](images/752-8.png)

Read the email file.

```bash
cat mailsent.txt
```

Recovered credentials:

```text
Username : daisa
Password : babygirl
```

![](images/752-9.png)

![](images/752-10.png)

---

# Web Enumeration

Visit the web applications.

```text
http://192.168.2.122
```

![](images/752-11.png)

```text
http://192.168.2.122:8000/
```

![](images/752-12.png)

---

## Directory Enumeration

```bash
gobuster dir -u http://192.168.2.122:8000 -w /usr/share/wordlists/dirb/common.txt --exclude-length 0
```

![](images/752-13.png)

Discovered the admin login page.

```text
http://192.168.2.122:8000/admin/
```

![](images/752-14.png)

Login using the credentials recovered from the SMB share.

```text
Email    : daisa@photographer.com
Password : babygirl
```

![](images/752-15.png)

Successfully authenticated.

![](images/752-16.png)

---

# Identify the CMS Version

Perform service detection on port **8000**.

```bash
nmap -sC -sV -p 8000 192.168.2.122
```

![](images/752-17.png)

Search for a public exploit corresponding to the detected CMS version.

Reference:

```text
https://www.exploit-db.com/exploits/48706
```

---

# Reverse Shell

Create a PHP reverse shell.

```bash
nano photographer.php.jpg
```

Insert your PHP reverse shell payload and update the attacker's IP address and listening port.

Upload the file through the CMS.

![](images/752-18.png)

Click **Import** to upload the image.

![](images/752-19.png)

---

## Bypass the Upload Filter

Intercept the upload request using Burp Suite.

![](images/752-20.png)

Send the request to **Repeater**.

![](images/752-21.png)

Modify the request according to the Exploit-DB technique and resend it.

![](images/752-22.png)

---

## Verify File Upload

Browse to the content directory.

```text
http://192.168.2.122:8000/content/
```

![](images/752-23.png)

---

## Start the Listener

```bash
nc -nlvp 443
```

Access the uploaded PHP file.

![](images/752-24.png)

A reverse shell is received.

![](images/752-25.png)

---

# Key Learning

- Enumerate SMB shares for sensitive files.
- Credentials discovered on SMB can often be reused for web applications.
- Identify CMS versions before searching for public exploits.
- Upload restrictions can sometimes be bypassed by modifying HTTP requests with Burp Suite.
- PHP reverse shells can be executed after a successful unrestricted file upload.
- Always verify the upload location before executing the payload.

---

# Summary

Initial access was achieved by enumerating anonymous SMB shares, where an email containing valid application credentials was discovered. These credentials allowed authentication to the CMS administration panel running on port **8000**. After identifying the CMS version, a known unrestricted file upload vulnerability was used. By modifying the upload request with Burp Suite, a PHP reverse shell was successfully uploaded and executed from the content directory, resulting in remote code execution and an interactive shell on the target system.